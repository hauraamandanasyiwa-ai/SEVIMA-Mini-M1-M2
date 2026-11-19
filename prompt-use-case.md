# PROMPT ENGINEERING — USE CASE DIAGRAM SEVIMA-MINI

> **Catatan Revisi Struktur:** Kode Mermaid Use Case Diagram yang sebelumnya berada di file ini telah dipindahkan dan disempurnakan ke [`diagrams/usecase-diagram.mmd`](../diagrams/usecase-diagram.mmd) sesuai struktur repository pada panduan tugas.

## Prompt Final — Generasi Use Case Diagram (Mermaid)

```text
# Role
Anda adalah system analyst berpengalaman dalam pemodelan UML.

# Konteks
Sistem informasi akademik SEVIMA-Mini yang memiliki 6 modul utama: PMB, KRS, Jadwal dan Ruangan, Nilai dan Transkrip, Keuangan/SPP, serta Presensi QR Code.

# Aktor
- Calon Mahasiswa (ProspectiveStudent)
- Mahasiswa (Student)
- Dosen / Dosen PA (Lecturer)
- Admin Akademik (Admin)
- Sistem PMB (aktor sistem)

# Use Case Utama & Relasi
- Modul PMB: Daftar Kuliah / PMB («include» Unggah Berkas Pendaftaran), Melihat Status Seleksi, Verifikasi Berkas PMB («extend» Generate NIM Otomatis oleh Sistem PMB).
- Modul KRS: Isi KRS («include» Validasi Prasyarat & Batas SKS, «include» Cek Bentrok Jadwal, «include» Kirim Notifikasi ke Dosen PA), Hapus Mata Kuliah («extend» Isi KRS), Approve KRS.
- Modul Jadwal & Ruangan: Melihat Jadwal Kuliah, Mengelola Jadwal dan Ruangan.
- Modul Nilai & Transkrip: Input Nilai, Melihat Nilai dan IPK, Generate Transkrip PDF («extend» Melihat Nilai dan IPK).
- Modul Keuangan: Melihat Tagihan SPP, Melakukan Pembayaran SPP, Verifikasi Pembayaran.
- Modul Presensi QR: Scan QR Presensi («include» Validasi Jadwal Presensi), Generate QR Sesi Kuliah, Melihat Riwayat Presensi.

# Format Output
Buat dalam format Mermaid `flowchart LR` agar tampilan vertikal rapi dan garis tidak saling bertumpuk. Gunakan guillemet «include» dan «extend» pada panah putus-putus agar kompatibel di GitHub dan Mermaid Live Editor.
```
