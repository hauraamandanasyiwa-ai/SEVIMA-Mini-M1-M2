# AI USAGE DOCUMENTATION & PROMPT LOG (SEVIMA-MINI)

Dokumen ini disusun sesuai **AI-First Development Charter** dan mencantumkan **6 iterasi prompt terbaik** beserta perbandingan hasil AI vs revisi manual, alasan teknis revisi, serta refleksi **OPAR** (*Observe, Plan, Act, Reflect*).

- **Tools AI yang digunakan:** Gemini / LLM Assistant (generasi & audit diagram UML), Mermaid Live Editor (validasi render).
- **Kebijakan Privasi:** Tidak ada data pribadi/rahasia (NIK, NIM asli, nilai asli mahasiswa) yang dimasukkan ke dalam prompt.

---

## Prompt 1 — Generasi Class Diagram Awal (Iterasi 1)

### Prompt yang Digunakan
> Saya sedang mengembangkan sistem informasi akademik sederhana bernama SEVIMA-Mini menggunakan konsep Object Oriented Programming (OOP). Sistem memiliki enam modul utama: PMB, KRS, Jadwal dan Ruangan, Nilai dan Transkrip, Keuangan/SPP, dan Presensi QR Code.
>
> Berdasarkan dokumen analisis domain, buatlah UML Class Diagram Mermaid dengan ketentuan:
> - `User` sebagai abstract class dengan turunan `Student`, `Lecturer`, `Admin`, `ProspectiveStudent`.
> - `Enrollment` sebagai penghubung `Student` dan `Course`.
> - `Grade` terhubung ke `Enrollment`.
> - Terapkan SRP, visibility (`+`, `-`, `#`), dan tampilkan multiplicity.

### Hasil AI (Before) vs Revisi Manual (After)
- **Output AI:** Menghasilkan 13 class yang valid secara sintaks Mermaid, hierarki `User` benar, dan `Grade` sudah terhubung ke `Enrollment`.
- **Masalah yang Ditemukan saat Validasi:**
  1. **Semua relasi hanya memakai association biasa (`-->`)** — tidak ada `composition` (`*--`) maupun `aggregation` (`o--`).
  2. **`Lecturer`, `Admin`, dan `ProspectiveStudent` menjadi class yatim (*orphan*)** — hanya terhubung ke `User` tanpa relasi ke `Schedule`, `Enrollment`, atau modul PMB.
  3. **Prasyarat `Course` hanya berupa `-prasyarat: String`**, bukan *self-association* `Course --> Course`.
  4. **Duplikasi atribut** `-semester` dan `-tahunAkademik` pada `Enrollment` dan `AcademicPeriod` (bahkan tipe data `int` vs `String` tidak konsisten).
  5. **`Attendance` tidak terhubung ke `Schedule`**, padahal presensi QR wajib divalidasi terhadap jadwal kuliah.
- **Alasan Revisi:** Diperlukan iterasi prompt lanjutan dengan unsur *Role* dan *Constraint* relasi UML yang eksplisit agar desain memenuhi prinsip OOP dan rubrik tugas.

---

## Prompt 2 — Refinement Relasi UML (Composition, Aggregation & Self-Association)

### Prompt yang Digunakan
> Anda adalah senior software architect. Perbaiki Mermaid Class Diagram SEVIMA-Mini berikut:
> [paste class-diagram v1]
>
> Terapkan perbaikan berikut:
> 1. Gunakan Composition (`*--`) untuk relasi kepemilikan kuat (`Enrollment`–`Grade`, `Course`–`Schedule`, `ProspectiveStudent`–`PMBRegistration`).
> 2. Gunakan Aggregation (`o--`) untuk relasi lemah (`AcademicPeriod`–`Enrollment`, `Room`–`Schedule`).
> 3. Ubah prasyarat `Course` menjadi self-association `Course "0..*" --> "0..*" Course : prasyarat`.
> 4. Hubungkan `Lecturer` ke `Schedule` dan `Enrollment`, `Admin` ke `PMBRegistration`, serta `Attendance` ke `Schedule`.
> 5. Hapus duplikasi atribut `semester` dan `tahunAkademik` dari `Enrollment`.
> Output hanya kode Mermaid `classDiagram` yang valid.

### Hasil AI vs Revisi Manual
- **Output AI:** Seluruh relasi `composition` dan `aggregation` berhasil ditambahkan dan tidak ada lagi class yatim.
- **Revisi Manual:**
  - Mengecek arah belah ketupat (`*--` dan `o--`) agar simbol wadah berada di sisi induk (`Enrollment "1" *-- "0..1" Grade`, bukan terbalik).
  - Menambahkan method abstrak `+tampilkanDashboard()* String` di `User` agar pilar **Polymorphism** terlihat eksplisit di diagram.
  - Memastikan tidak ada *circular dependency* (seluruh arah panah ke `Enrollment`, `Schedule`, dan `Attendance` bersifat satu arah).

---

## Prompt 3 — Audit SOLID & Refactoring dengan Design Pattern

### Prompt yang Digunakan
> Sebagai principal software architect, audit Class Diagram SEVIMA-Mini berikut terhadap prinsip SOLID (khususnya SRP, OCP, dan DIP):
> [paste class-diagram v2]
>
> Identifikasi pelanggaran SRP/OCP/DIP, lalu refactor agar:
> - Validasi KRS (maks 24 SKS, cek prasyarat, cegah double enrollment, cek bentrok jadwal) menggunakan **Strategy Pattern** (`KRSService` + interface `IValidationRule`).
> - Transisi `ProspectiveStudent` lulus seleksi menjadi `Student` menggunakan **Factory Pattern** (`StudentFactory`).
> - Perhitungan IPS/IPK dan generate transkrip dipisah ke `TranscriptService`.
> - Pengiriman notifikasi diabstraksikan lewat `INotificationService`.
> Pastikan setiap class tetap ramping (< 10 method) dan bebas circular dependency.

### Hasil AI vs Revisi Manual
- **Output AI:** Memisahkan logika validasi ke 4 concrete strategy (`MaxSKSValidation`, `PrerequisiteValidation`, `ScheduleConflictValidation`, `DoubleEnrollmentValidation`), menambah `StudentFactory`, `TranscriptService`, dan `INotificationService`.
- **Revisi Manual:**
  - Menghapus method `tambahMataKuliah()`/`ubahMataKuliah()` dari class entitas `Course` dan `hitungTotalSKS()` dari `Enrollment` karena melanggar SRP (dipindahkan ke `KRSService`).
  - Memperbaiki sintaks tipe generik Mermaid dari `List<Schedule>` (yang sempat menyebabkan error parse) menjadi `List~Schedule~`.
  - Memverifikasi relasi `KRSService "1" o-- "1..*" IValidationRule` sebagai aggregation agar daftar rule dapat disuntikkan (*dependency injection*).

---

## Prompt 4 — Generasi Use Case Diagram & Perbaikan Layout

### Prompt yang Digunakan
> Anda adalah system analyst. Buat Use Case Diagram dalam format Mermaid `flowchart LR` untuk sistem akademik SEVIMA-Mini dengan aktor:
> `Calon Mahasiswa`, `Mahasiswa`, `Dosen / Dosen PA`, `Admin Akademik`, dan `Sistem PMB`.
>
> Kelompokkan use case ke dalam 6 modul (PMB, KRS, Jadwal & Ruangan, Nilai & Transkrip, Keuangan/SPP, Presensi QR) plus Autentikasi.
> Wajib sertakan relasi `«include»` dan `«extend»` yang akurat (misalnya Isi KRS include Validasi Prasyarat & Batas SKS, Cek Bentrok Jadwal, Kirim Notifikasi ke Dosen PA; Hapus MK extend Isi KRS; Generate Transkrip PDF extend Melihat Nilai dan IPK; Scan QR Presensi include Validasi Jadwal Presensi).

### Hasil AI vs Revisi Manual
- **Output AI (Iterasi Awal):** Awalnya dibuat dalam `flowchart TB` tanpa aktor `Sistem PMB` dan tanpa relasi `<<include>>`/`<<extend>>`, serta garis antar aktor dan use case saling bertumpuk melebar.
- **Revisi Manual:**
  - Memindahkan file dari `prompts/prompt-use-case.md` ke lokasi yang benar sesuai template: `diagrams/usecase-diagram.mmd`.
  - Mengubah orientasi ke `flowchart LR` agar jauh lebih rapi dibaca.
  - Menggunakan tanda guillemet `«include»` dan `«extend»` agar tidak terpotong sebagai tag HTML oleh renderer Mermaid.
  - Menambahkan aktor `Sistem PMB` dan menyinkronkan seluruh nama use case dengan method di Class Diagram.

---

## Prompt 5 — Sequence Diagram Proses PMB (Dengan Alternate Flow)

### Prompt yang Digunakan
> Anda adalah arsitek sistem. Buat Mermaid `sequenceDiagram` untuk skenario Proses Pendaftaran Mahasiswa Baru (PMB) pada SEVIMA-Mini.
> Partisipan harus sinkron dengan Class Diagram: `ProspectiveStudent`, `Sistem PMB`, `PMBRegistration`, `Admin Akademik`, `StudentFactory`, `Database`, dan `NotificationService`.
>
> Alur utama: daftar PMB → unggah berkas → verifikasi berkas oleh Admin → input hasil seleksi → lulus → `StudentFactory.createStudentFromPMB()` + `generateNIM()` → notifikasi ke calon mahasiswa.
> Wajib sertakan blok `alt/else` untuk error handling: (1) berkas tidak lengkap/revisi, dan (2) tidak lulus seleksi.

### Hasil AI vs Revisi Manual
- **Output AI:** Menghasilkan alur utama dan dua cabang `alt`, namun terdapat kesalahan aktivasi lifeline (`->>+` dibuka sebelum `alt` dan `-->>-` hanya ditutup di satu cabang), yang rawan error saat dirender.
- **Revisi Manual:**
  - Merapikan seluruh pasangan aktivasi `+`/`-` agar selalu simetris di luar percabangan `alt`.
  - Memastikan setiap cabang `alt` mengirim notifikasi balik ke `ProspectiveStudent` lewat `NotificationService`.

---

## Prompt 6 — Sequence Diagram Validasi KRS (Strategy Pattern & Error Handling)

### Prompt yang Digunakan
> Buat Mermaid `sequenceDiagram` untuk proses Validasi KRS pada SEVIMA-Mini yang sinkron dengan Class Diagram.
> Partisipan: `Student`, `Sistem KRS`, `KRSService`, `PrerequisiteValidation`, `DoubleEnrollmentValidation`, `MaxSKSValidation`, `ScheduleConflictValidation`, `Database`, `NotificationService`, `Lecturer (Dosen PA)`.
>
> Sertakan:
> 1. Pengecekan `AcademicPeriod` aktif dan status `Payment` SPP lunas.
> 2. Loop validasi setiap mata kuliah (prasyarat, double enrollment, batas maks 24 SKS, bentrok jadwal).
> 3. Jika lolos semua validasi, simpan `Enrollment` berstatus `Pending Approval` dan kirim notifikasi ke Dosen PA.
> 4. Wajib lengkapi `alt/else` untuk setiap kondisi gagal.

### Hasil AI vs Revisi Manual
- **Output AI:** Menghasilkan alur 4 validasi inti, namun belum mengecek status periode KRS aktif dan status lunas SPP sebelum masuk loop validasi.
- **Revisi Manual:**
  - Menambahkan pengecekan prasyarat administratif (`cekPeriodeAktif` dan `cekStatusPembayaranSPP`) di awal sesuai aturan bisnis #1 dan #12 pada [analisis-domain.md](../docs/analisis-domain.md).
  - Menyinkronkan nama method persis seperti pada [class-diagram.mmd](../diagrams/class-diagram.mmd) (`submitKRS`, `validate`, `isBentrokDengan`, `sendNotification`).

---

## Catatan OPAR (Observe – Plan – Act – Reflect)

| Tahap | Ringkasan |
| :--- | :--- |
| **Observe** | Output AI pertama terlihat rapi secara visual, namun saat diaudit ternyata menyamaratakan semua relasi menjadi association biasa (`-->`), meninggalkan class yatim (`Lecturer`, `Admin`), membuat atribut duplikat (`semester`/`tahunAkademik`), dan tidak menerapkan *design pattern* maupun *alternate flow* secara lengkap. |
| **Plan** | Menyusun prompt terstruktur dengan 5 unsur (*Role, Context, Components, Constraints, Output Format*) dan melakukan iterasi bertahap: perbaikan relasi UML → audit SOLID & Pattern → sinkronisasi Use Case & Sequence Diagram. |
| **Act** | Menjalankan 6 iterasi prompt, merevisi hasil AI secara manual pada setiap tahap (arah composition/aggregation, sintaks generik `List~T~`, keseimbangan lifeline `+`/`-`, serta kesamaan nama class/method antar 3 jenis diagram). |
| **Reflect** | AI sangat cepat menghasilkan *draft* struktur awal, tetapi **validasi arsitektur tetap membutuhkan pemahaman manusia**—terutama dalam membedakan *composition* vs *aggregation*, memecah tanggung jawab (SRP), dan memastikan sinkronisasi lintas diagram sebelum masuk tahap coding Minggu 3–16. |
