# 📊 OpenProject Excel Integration (OpenProjectXLS_API.xlsm)

Dokumen ini berisi petunjuk lengkap mengenai cara menggunakan file **OpenProjectXLS_API.xlsm** untuk mengimpor, mengekspor, dan memperbarui data Work Packages dari OpenProject melalui API.

---

## 📥 1. Download File

Unduh file `OpenProjectXLS_API.xlsm` dari link berikut:

> 🔗 **[Download OpenProjectXLS_API.xlsm](#)**  
> _(Ganti `#` dengan link download asli)_

---

## ⚙️ 2. Persiapan File

Setelah file berhasil diunduh:

1. Klik kanan pada file `OpenProjectXLS_API.xlsm` → pilih **Properties**.
2. Pada tab **General**, centang opsi **Unblock** pada bagian Security.
3. Klik **Apply** lalu **OK**.

---

## 🔐 3. Konfigurasi Koneksi

Buka file `OpenProjectXLS_API.xlsm`, lalu isi field berikut:

- **Instance URL**:  
  Contoh: `https://project.nama_perusahaan.com/`
  
- **Project**:  
  Gunakan identifier dari URL project, contoh:  
  Jika URL adalah `https://project.nama_perusahaan.com/projects/it-project`, maka `Project` = `it-project`

- **API Token**:  
  Buat token baru dari OpenProject:
  
  - Klik logo profil (kanan atas) → **Account Settings**
  - Pilih menu kiri **Access tokens**
  - Masukkan nama token dan klik **Create**
  - Salin token tersebut ke field `API Token`

Setelah semua terisi, klik **Accept** untuk melakukan koneksi.

---

## ✅ 4. Verifikasi Koneksi

Jika koneksi berhasil:

- Menu tambahan akan muncul secara otomatis di Excel.
- Gunakan shortcut `Ctrl + B` untuk membuka menu **Workpackages**.

---

## 🔁 5. Menu Workpackages

Terdapat 3 fitur utama dalam menu Workpackages:

### a) **Download Workpackages**
- Menarik seluruh data workpackages dari project.

### b) **Upload/Update Workpackages**
- Mengimpor atau memperbarui data workpackages ke dalam project.

### c) **Configuration**
- Membuka kembali menu konfigurasi koneksi (Instance URL, Project, API Token).

---

## 🧩 6. Struktur Kolom & Data

- Kolom-kolom yang tersedia di sheet `Workpackages` dapat disesuaikan.
- Lihat sheet `Attributes` untuk mengetahui label dan struktur field yang valid.
- Gunakan label kolom sesuai dengan definisi di sheet `Attributes`.

> ⚠️ **Catatan:**  
> Jangan mengubah kolom berikut saat melakukan update:
> - `Status`
> - `Lock version`
> - `ID`

---

## 🔄 7. Pindah ke Project Lain

Untuk beralih ke project berbeda:

1. Ganti `Project` pada menu konfigurasi koneksi.
2. Lihat identifier project di sheet `Projects`.

---

## 📤 8. Download & Update Workpackages

1. Tekan `Ctrl + B` untuk membuka menu.
2. Klik **Download workpackages** untuk menarik data dari server.
3. Edit data sesuai kebutuhan.
4. Untuk mengunggah ulang:
   - Tekan `Ctrl + B`
   - Pilih **Upload/update workpackages**

---

## 📝 Lisensi

File ini disediakan untuk penggunaan internal perusahaan. Harap tidak menyebarkan tanpa izin.

---

## 📧 Kontak

Jika terdapat kendala atau pertanyaan, hubungi tim IT/PMO atau developer internal yang menyediakan integrasi ini.
