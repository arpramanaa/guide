# 📊 OpenProject Excel Integration (`OpenProjectXLS_API.xlsm`)

Dokumen ini berisi petunjuk lengkap mengenai cara menggunakan file **OpenProjectXLS_API.xlsm** untuk mengimpor, mengekspor, dan memperbarui data *Work Packages* dari OpenProject melalui API.

---

## 📥 1. Download File

Unduh file `OpenProjectXLS_API.xlsm` dari link berikut:

> 🔗 **[Download OpenProjectXLS_API.xlsm](https://github.com/arpramanaa/resources/tree/main/openproject)**

---

## ⚙️ 2. Persiapan File

<img src="https://github.com/user-attachments/assets/f9e55f71-0f17-4f5e-82fc-affb0df22729" alt="Unblock File" width="553" height="414" />

Setelah file berhasil diunduh:

1. Klik kanan pada file `OpenProjectXLS_API.xlsm` → pilih **Properties**.
2. Pada tab **General**, centang opsi **Unblock** pada bagian Security.
3. Klik **Apply** lalu **OK**.

---

## 🔐 3. Konfigurasi Koneksi

<img src="https://github.com/user-attachments/assets/60e382a5-ec23-4cce-9a1d-376ad605aaef" alt="Form Konfigurasi" width="553" height="414" />

Buka file `OpenProjectXLS_API.xlsm`, lalu isi field berikut:

- **Instance URL**  
  Contoh: `https://project.nama_perusahaan.com/`

- **Project**  
  <img src="https://github.com/user-attachments/assets/2d6d87f6-402f-4d14-92d4-4a2379ea24aa" alt="Project Identifier" width="417" height="311" />

  Gunakan identifier dari URL project, contoh:  
  Jika URL adalah `https://project.nama_perusahaan.com/projects/it-project`, maka `Project` = `it-project`

- **API Token**  
  <img src="https://github.com/user-attachments/assets/1613098d-11b4-4012-b502-2af65a839b7f" alt="Generate API Token" width="442" height="330" />

  Langkah membuat token:
  1. Klik logo profil (kanan atas) → **Account Settings**
  2. Pilih menu kiri **Access tokens**
  3. Masukkan nama token dan klik **Create**
  4. Salin token tersebut ke field `API Token`

Setelah semua terisi, klik **Accept** untuk melakukan koneksi.

---

## ✅ 4. Verifikasi Koneksi

Jika koneksi berhasil:

- Menu tambahan akan muncul secara otomatis di Excel.
- Gunakan shortcut `Ctrl + B` untuk membuka menu **Workpackages**.

---

## 🔁 5. Menu Workpackages

<img src="https://github.com/user-attachments/assets/d5fe3ed8-2b3f-46ad-bbf1-d9da0ca3f06c" alt="Menu Workpackages" width="553" height="414" />

Menu ini memiliki 3 fitur utama:

### a) **Download Workpackages**
Menarik seluruh data *workpackages* dari project.

### b) **Upload/Update Workpackages**
Mengimpor atau memperbarui data *workpackages* ke dalam project.

### c) **Configuration**
Membuka kembali menu konfigurasi koneksi.

---

## 🧩 6. Struktur Kolom & Data

<img src="https://github.com/user-attachments/assets/2303902e-5f1a-4762-89f6-bf9dc8c29b28" alt="Sheet Attributes" width="377" height="281" />

- Kolom-kolom di sheet `Workpackages` dapat disesuaikan.
- Sheet `Attributes` memberikan informasi struktur dan label field yang valid.
- Gunakan label kolom sesuai dengan definisi di sheet `Attributes`.

> ⚠️ **Catatan:**  
> Jangan mengubah kolom berikut saat melakukan update:
> - `Status`
> - `Lock version`
> - `ID`

---

## 🔄 7. Pindah ke Project Lain

Untuk beralih ke project yang berbeda:

1. Ganti nilai `Project` di menu konfigurasi.
2. Lihat identifier project di sheet `Projects`.

---

## 📤 8. Download & Update Workpackages

1. Tekan `Ctrl + B` untuk membuka menu.
2. Klik **Download workpackages** untuk mengambil data dari server.
3. Edit data yang diperlukan.
4. Tekan `Ctrl + B` kembali, lalu klik **Upload/update workpackages** untuk mengunggah data terbaru.

---

## 📧 Kontak

Jika terdapat kendala atau pertanyaan, silakan hubungi tim **IT/PMO** atau developer internal yang bertanggung jawab atas integrasi ini.
