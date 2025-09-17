# Panduan Instalasi Kubernetes (kubeadm) dengan containerd dengan Ubuntu/Debian

Dokumen ini berisi langkah-langkah end-to-end untuk membuat cluster Kubernetes menggunakan **kubeadm** dan **containerd** (control-plane + worker).

---

## Syarat

* Akses `sudo` di semua node (control-plane & worker).
* Kernel modern (Linux) dan koneksi jaringan antar node.
* Nonaktifkan swap pada semua node.
* Tested: Ubuntu 22.04/23.04 (apt command).

---

## 1. Persiapan host (jalankan di semua node)

```bash
# update & nonaktifkan swap
sudo apt update && sudo apt upgrade -y
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# module kernel utk CNI
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl networking
cat <<EOF | sudo tee /etc/sysctl.d/99-k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

---

## 2. Install containerd & konfigurasikan systemd cgroup (jalankan di semua node)

```bash
# install dependencies
sudo apt install -y ca-certificates curl gnupg lsb-release

# pakai repo docker untuk containerd.io (opsional)
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install -y containerd.io

# buat config dan set systemd cgroup
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

> Kenapa `SystemdCgroup = true`? Agar kubelet (yang default menggunakan systemd cgroup) sinkron dengan runtime.

---

## 3. Pasang CNI plugin binaries (/opt/cni/bin)

Beberapa CNI (flannel, calico) mengandalkan plugin di `/opt/cni/bin`. Jika kosong, pod sandbox gagal.

```bash
sudo mkdir -p /opt/cni/bin
CNI_VER=v1.4.0
curl -L -o cni-plugins.tgz https://github.com/containernetworking/plugins/releases/download/${CNI_VER}/cni-plugins-linux-amd64-${CNI_VER}.tgz
sudo tar -C /opt/cni/bin -xzf cni-plugins.tgz
rm -f cni-plugins.tgz
```

---

## 4. (Opsional tapi berguna) Pasang `crictl` untuk debugging CRI

Pilih versi `crictl` yang kompatibel dengan Kubernetes target (contoh v1.30.0):

```bash
CRICTL_VER="v1.30.0"
curl -L -o crictl.tar.gz https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VER}/crictl-${CRICTL_VER}-linux-amd64.tar.gz
sudo tar -C /usr/local/bin -xzf crictl.tar.gz
rm -f crictl.tar.gz
```

Jika menemukan `permission denied` untuk socket, jalankan `sudo crictl info` atau buat `/etc/crictl.yaml` dengan `runtime-endpoint: unix:///run/containerd/containerd.sock`.

---

## 5. Install kubeadm, kubelet, kubectl (pin versi) — jalankan di semua node

Contoh pakai repositori Kubernetes `v1.30` (sesuaikan versi yang ingin dipakai):

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo mkdir -p /etc/apt/keyrings
curl -fsSLo /etc/apt/keyrings/kubernetes-apt-keyring.gpg https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
# ganti 1.30.14-1.1 sesuai versi target
sudo apt-get install -y kubeadm=1.30.14-1.1 kubelet=1.30.14-1.1 kubectl=1.30.14-1.1
sudo apt-mark hold kubeadm kubelet kubectl
```

> Jika menemukan versi kubelet terlalu tinggi vs kubeadm -> downgrade kubelet ke versi kubeadm.

---

## 6. Init control-plane (hanya di control-plane node)

Pilih pod-network-cidr sesuai CNI (Flannel = 10.244.0.0/16). Pastikan containerd berjalan.

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket unix:///run/containerd/containerd.sock
```

Jika sukses, catat perintah `kubeadm join ...` yang keluar (akan digunakan worker node).

---

## 7. Konfigurasi kubectl untuk user non-root (control-plane)

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl cluster-info
kubectl get nodes
```

---

## 8. Pasang CNI (contoh Flannel)

Jalankan setelah kubeadm init selesai dan kubeconfig sudah diatur:

```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

Atau pakai Calico (lebih feature-rich):

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml
```

Tunggu `kubectl get pods -n kube-system` sampai `kube-flannel-ds-*` dan `coredns` berstatus `Running`.

---

## 9. Join worker node(s)

Jalankan perintah `kubeadm join` yang diberikan di output `kubeadm init` pada tiap worker. Contoh bila lupa:

```bash
# generate ulang join command di control-plane
sudo kubeadm token create --print-join-command
```

Di worker:

```bash
sudo kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --cri-socket unix:///run/containerd/containerd.sock
```

---

## 10. Verifikasi akhir

```bash
kubectl get nodes
kubectl get pods -A
kubectl get pods -n kube-system -o wide
```

Semua node harus `Ready` dan pod `kube-system` harus `Running`.

---

## 11. Troubleshooting singkat (masalah umum)

* `tls: failed to verify certificate` → copy `/etc/kubernetes/admin.conf` ke `$HOME/.kube/config` dan set ownership.
* `failed to find plugin "loopback" in path [/opt/cni/bin]` → pasang CNI plugin (lihat langkah 3).
* `permission denied` on `/run/containerd/containerd.sock` → jalankan `sudo crictl info` atau set `runtime-endpoint` di `/etc/crictl.yaml`; untuk non-root tambahkan user ke group yang punya akses socket.
* `Port 10257 in use` → ada sisa proses Kubernetes; jalankan `sudo kubeadm reset -f`, stop kubelet, hapus sisa `/etc/kubernetes` dan direktori state.
* `kubelet version > control plane` → downgrade kubelet ke versi kubeadm / control plane.

---

## 12. Reset / cleanup (hapus state cluster di node ini)

**PERINGATAN**: ini menghapus semua state Kubernetes pada node.

```bash
sudo kubeadm reset -f
sudo systemctl stop kubelet
sudo systemctl disable kubelet
sudo rm -rf /etc/cni/net.d /var/lib/cni/ /var/lib/kubelet/* /etc/kubernetes/ /var/lib/etcd
sudo iptables -F
sudo systemctl restart containerd
```

---

## 13. Test deployment sederhana

```bash
kubectl create deployment nginx --image=nginx:1.24
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get svc
kubectl get pods -o wide
```

---

## 14. Catatan & rekomendasi

* Selalu gunakan versi kubeadm/kubelet/kubectl yang cocok (minor skew ±0-1). Pin paket dengan `apt-mark hold`.
* Untuk production, pertimbangkan HA control-plane, persistent etcd, dan network/security hardening.
* Backup `/etc/kubernetes` dan etcd jika penting.

---

