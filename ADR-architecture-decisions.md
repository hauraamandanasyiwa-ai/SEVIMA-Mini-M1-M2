# Architecture Decision Record (ADR) — Desain Sistem Akademik SEVIMA-Mini

- **Proyek:** SEVIMA-Mini (Sistem Informasi Akademik Berbasis OOP)
- **Status:** Accepted
- **Tanggal:** 2026-11-19
- **Lingkup:** Desain Arsitektur UML (Minggu 1–2) sebagai fondasi implementasi Minggu 3–16

---

## 1. Konteks Arsitektur

SEVIMA-Mini mengelola enam modul akademik inti: Penerimaan Mahasiswa Baru (PMB), Kartu Rencana Studi (KRS), Jadwal & Ruangan, Nilai & Transkrip, Keuangan/SPP, serta Presensi QR Code. Tanpa desain arsitektur yang disiplin sebelum tahap implementasi, entitas seperti `Student` dan `Enrollment` berisiko berkembang menjadi *God Object* yang memiliki puluhan method, relasi saling bergantung melingkar (*circular dependency*), serta sulit dikembangkan saat aturan bisnis kampus bertambah.

---

## 2. Keputusan Arsitektur Utama

### ADR-01: Abstraksi Hierarki `User` & Transisi `ProspectiveStudent` ke `Student` (Factory Pattern)
- **Keputusan:** Menjadikan `User` sebagai `<<abstract>>` class dengan empat turunan (`ProspectiveStudent`, `Student`, `Lecturer`, `Admin`) serta menerapkan **Factory Pattern** (`StudentFactory`) untuk membuat objek `Student` baru saat calon mahasiswa dinyatakan lulus seleksi PMB.
- **Alasan:** Tidak ada pengguna di dunia nyata yang hanya berstatus "User" tanpa peran. Atribut bersama (`id`, `username`, `email`) dan autentikasi (`login()`, `logout()`) ditarik ke induk, sementara `-password` dibuat `private` untuk enkapsulasi. Saat `ProspectiveStudent` diterima, dalam OOP sebuah objek tidak dapat berganti class secara langsung; oleh karena itu `StudentFactory.createStudentFromPMB()` membuat instance `Student` baru, menghasilkan NIM otomatis, dan menyimpan keterkaitannya dengan data pendaftaran.
- **Trade-off:** Menambah satu class `StudentFactory` dan penyimpanan referensi pendaftaran, namun menjaga tipe objek tetap aman (*type-safe*), jejak historis PMB terjaga, dan logika format NIM terpusat di satu tempat.

### ADR-02: Pemecahan Relasi Many-to-Many lewat `Enrollment` & Pelekatan `Grade` dengan Composition
- **Keputusan:** Menggunakan `Enrollment` sebagai class asosiasi antara `Student`, `Course`, dan `AcademicPeriod`, lalu menghubungkan `Grade` ke `Enrollment` dengan **Composition (`*--`)**, serta `Course` ke `Schedule` dengan **Composition (`*--`)**.
- **Alasan:** Satu mahasiswa mengambil banyak mata kuliah dan satu mata kuliah diambil banyak mahasiswa. `Enrollment` menyimpan konteks pengambilan (`status`, `tanggalSubmit`). `Grade` tidak dihubungkan langsung ke `Student` karena nilai selalu spesifik pada satu mahasiswa, satu mata kuliah, dan satu periode tertentu. Jika `Enrollment` dibatalkan/dihapus, `Grade` tidak lagi memiliki makna (*composition*). Sebaliknya, `AcademicPeriod`–`Enrollment` dan `Room`–`Schedule` menggunakan **Aggregation (`o--`)** karena ruangan atau data historis tetap eksis secara independen.
- **Trade-off:** Query transkrip memerlukan navigasi `Student` → `Enrollment` → `Grade`, sedikit lebih dalam dibanding relasi langsung, namun menghilangkan anomali data nilai yatim dan normalisasi semester/tahun akademik tetap bersih tanpa duplikasi.

### ADR-03: Self-Association untuk Prasyarat Mata Kuliah (`Course --> Course`)
- **Keputusan:** Memodelkan prasyarat mata kuliah sebagai *self-association* `Course "0..*" --> "0..*" Course : prasyarat`, bukan atribut `-prasyarat: String`.
- **Alasan:** Satu mata kuliah dapat memiliki nol, satu, atau lebih mata kuliah prasyarat. Relasi antar objek `Course` memungkinkan `PrerequisiteValidation` memeriksa kelulusan mata kuliah secara terprogram dan akurat dibanding parsing teks `String`.
- **Trade-off:** Membutuhkan validasi pencegahan siklus prasyarat (A butuh B, B butuh A) pada level input kurikulum, tetapi secara drastis meningkatkan integritas data akademik.

### ADR-04: Pemisahan Validasi KRS dengan Strategy Pattern & Abstraksi Notifikasi (SOLID: SRP, OCP, DIP)
- **Keputusan:** Memisahkan aturan validasi KRS dari `Student`/`Enrollment` ke `KRSService` yang menerapkan **Strategy Pattern** melalui interface `IValidationRule` (`MaxSKSValidation`, `PrerequisiteValidation`, `ScheduleConflictValidation`, `DoubleEnrollmentValidation`), memisahkan perhitungan IPS/IPK ke `TranscriptService`, dan mengabstraksikan pengiriman pesan lewat `INotificationService`.
- **Alasan:**
  - **SRP:** `Student` hanya memegang data diri & aksi dasar mahasiswa; perubahan aturan batas SKS atau format PDF transkrip tidak memaksa perubahan pada class `Student`.
  - **OCP:** Bila kampus menambah aturan baru (misal: batas SKS berdasarkan IPS semester lalu atau kuota kelas), cukup menambah class baru implementasi `IValidationRule` tanpa memodifikasi kode inti `KRSService`.
  - **DIP:** `KRSService` bergantung pada abstraksi `IValidationRule` dan `INotificationService`, sehingga mudah diuji (*unit testing* dengan *mock*) dan mudah diganti kanal notifikasinya (Email, WhatsApp, Push Notification).
- **Trade-off:** Jumlah class bertambah dari 13 menjadi 22 class sehingga diagram awal tampak lebih detail, namun setiap class tetap ramping (2–4 method), bebas *God Object*, bebas *circular dependency*, dan jauh lebih siap untuk implementasi bertahap Minggu 3–16.

---

## 3. Ringkasan Pola Desain (Design Patterns) yang Diterapkan

| Design Pattern | Komponen | Manfaat Utama |
| :--- | :--- | :--- |
| **Strategy Pattern** | `KRSService`, `IValidationRule`, dan 4 concrete validator | Aturan validasi KRS modular, dapat ditambah tanpa ubah kode inti (OCP) |
| **Factory Pattern** | `StudentFactory` | Enkapsulasi pembuatan `Student` baru & generasi NIM otomatis dari hasil PMB |
| **Dependency Inversion / Service Abstraction** | `INotificationService`, `NotificationService`, `TranscriptService` | Memisahkan urusan notifikasi & pelaporan PDF dari entitas domain inti (SRP & DIP) |
