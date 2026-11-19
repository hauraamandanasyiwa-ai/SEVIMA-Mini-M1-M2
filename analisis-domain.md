# ANALISIS DOMAIN SEVIMA-MINI

## 1. Gambaran Umum Sistem

**SEVIMA-Mini** merupakan sistem informasi akademik sederhana untuk perguruan tinggi yang dirancang menggunakan prinsip **Object-Oriented Programming (OOP)** dan **SOLID**. Sistem ini menjadi fondasi arsitektur dari fase desain (Minggu 1–2) hingga fase implementasi (Minggu 3–16).

Sistem terdiri dari enam modul utama:
1. **PMB (Penerimaan Mahasiswa Baru):** pendaftaran calon mahasiswa, unggah & verifikasi berkas, hasil seleksi, serta pembuatan NIM otomatis.
2. **KRS (Kartu Rencana Studi):** pemilihan mata kuliah dengan validasi prasyarat, batas maksimal 24 SKS, pencegahan *double enrollment*, deteksi bentrok jadwal, serta persetujuan Dosen PA.
3. **Jadwal dan Ruangan:** pengelolaan jadwal perkuliahan, pengampu mata kuliah, serta ketersediaan ruangan.
4. **Nilai dan Transkrip:** input nilai oleh dosen, konversi nilai angka ke huruf/bobot, perhitungan IPS/IPK, dan generate transkrip PDF.
5. **Keuangan / Pembayaran SPP:** pencatatan tagihan SPP per periode akademik beserta status lunas atau menunggak.
6. **Presensi QR Code:** pencatatan kehadiran mahasiswa per sesi jadwal kuliah melalui pemindaian QR Code.

---

## 2. Identifikasi Kandidat Class

### 2.1 Hierarki User (Abstraction, Inheritance, Polymorphism)
- **`User` (`<<abstract>>`):** induk seluruh pengguna sistem (`#id`, `#username`, `-password`, `#email`; `+login()`, `+logout()`, `+tampilkanDashboard()*`).
- **`ProspectiveStudent`:** calon mahasiswa baru (`-nomorPendaftaran`, `-statusSeleksi`; `+daftarPMB()`, `+lihatStatusSeleksi()`).
- **`Student`:** mahasiswa aktif (`-nim`, `-programStudi`, `-semester`, `-ipk`; `+lihatJadwal()`, `+lakukanPresensi()`).
- **`Lecturer`:** dosen pengampu dan Dosen PA (`-nidn`, `-departemen`; `+approveKRS()`, `+inputNilai()`, `+lihatJadwalMengajar()`).
- **`Admin`:** pengelola administrasi akademik & PMB (`-kodeAdmin`; `+verifikasiBerkasPMB()`, `+kelolaPeriode()`, `+kelolaJadwal()`).

### 2.2 Entitas Inti Akademik & PMB
- **`PMBRegistration`:** mencatat pendaftaran, kelengkapan berkas, dan status verifikasi calon mahasiswa.
- **`AcademicPeriod`:** menyimpan periode semester aktif (`-idPeriode`, `-semester`, `-tahunAkademik`, `-statusPeriode`).
- **`Course`:** informasi mata kuliah (`-kode`, `-nama`, `-sks`) dengan *self-association* ke `Course` lain sebagai daftar prasyarat.
- **`Enrollment`:** class perantara (pemecah relasi *many-to-many* antara `Student` dan `Course`) dalam satu `AcademicPeriod`, menyimpan `-idEnrollment`, `-tanggalSubmit`, dan `-status` (`Draft`, `Pending Approval`, `Approved`, `Rejected`).
- **`Grade`:** menyimpan `-nilaiAngka`, `-nilaiHuruf`, serta konversi bobot nilai yang melekat kuat pada satu `Enrollment`.
- **`Schedule`:** jadwal sesi kelas (`-hari`, `-jamMulai`, `-jamSelesai`) dengan deteksi bentrok waktu.
- **`Room`:** data ruang kuliah (`-kodeRuangan`, `-namaRuangan`, `-kapasitas`) beserta pengecekan ketersediaan.
- **`Payment`:** tagihan dan pembayaran SPP mahasiswa per `AcademicPeriod` (`-jumlah`, `-tanggalBayar`, `-statusPembayaran`).
- **`Attendance`:** catatan kehadiran mahasiswa pada sesi `Schedule` tertentu melalui QR Code.

### 2.3 Service & Design Pattern (SRP, OCP, DIP)
Untuk mencegah *God Object* pada `Student` maupun `Enrollment`, tanggung jawab validasi, pembuatan objek, notifikasi, dan pelaporan dipecah menjadi:
- **`StudentFactory` (Factory Pattern):** membuat objek `Student` baru beserta generasi NIM otomatis ketika `ProspectiveStudent` lulus seleksi.
- **`KRSService` + `IValidationRule` (Strategy Pattern):** mengeksekusi aturan validasi KRS secara modular (`MaxSKSValidation`, `PrerequisiteValidation`, `ScheduleConflictValidation`, `DoubleEnrollmentValidation`).
- **`INotificationService` / `NotificationService` (Dependency Inversion):** abstraksi pengiriman notifikasi ke Calon Mahasiswa, Mahasiswa, maupun Dosen PA.
- **`TranscriptService` (Single Responsibility):** menghitung IPS/IPK dan menghasilkan transkrip PDF dari data `Enrollment` dan `Grade`.

---

## 3. Identifikasi Hubungan Antarclass

| Jenis Relasi | Simbol UML | Contoh di SEVIMA-Mini | Alasan Desain |
| :--- | :---: | :--- | :--- |
| **Inheritance** | `<\|--` | `User` ← `Student`, `Lecturer`, `Admin`, `ProspectiveStudent` | Hubungan *is-a*; atribut & autentikasi bersama naik ke induk abstrak |
| **Realization** | `<\|..` | `IValidationRule` ← `MaxSKSValidation`, dll.; `INotificationService` ← `NotificationService` | Kontrak interface untuk OCP & DIP |
| **Composition** | `*--` | `Enrollment` ◆— `Grade`, `Course` ◆— `Schedule`, `ProspectiveStudent` ◆— `PMBRegistration` | Relasi kuat: bagian tidak bermakna/ikut terhapus bila wadahnya dihapus |
| **Aggregation** | `o--` | `AcademicPeriod` ◇— `Enrollment`, `Room` ◇— `Schedule`, `KRSService` ◇— `IValidationRule` | Relasi lemah: bagian tetap bisa eksis meski wadah berubah/ditutup |
| **Association** | `-->` | `Student` → `Enrollment`, `Course` → `Course` (prasyarat), `Lecturer` → `Schedule`/`Enrollment`, `Schedule` → `Attendance` | Hubungan antar entitas yang saling berinteraksi dengan kardinalitas jelas |
| **Dependency** | `..>` | `StudentFactory` ⇢ `Student`, `KRSService` ⇢ `INotificationService`, `TranscriptService` ⇢ `Enrollment` | Service memanfaatkan entitas/abstraksi tanpa kepemilikan siklus hidup |

---

## 4. Aturan Bisnis Utama

1. Mahasiswa hanya dapat mengisi KRS saat `AcademicPeriod` berstatus aktif dan `Payment` SPP periode tersebut berstatus lunas.
2. Batas maksimal pengambilan KRS adalah **24 SKS** per semester (`MaxSKSValidation`).
3. Mahasiswa wajib lulus seluruh mata kuliah prasyarat sebelum mengambil mata kuliah lanjutan (`PrerequisiteValidation`).
4. Sistem mencegah *double enrollment* untuk mata kuliah yang sama pada periode yang sama (`DoubleEnrollmentValidation`).
5. Sistem mencegah bentrok jadwal perkuliahan (`ScheduleConflictValidation`).
6. Jika seluruh validasi KRS lolos, status `Enrollment` berubah menjadi `Pending Approval` dan notifikasi dikirim ke Dosen PA (`Lecturer`).
7. Nilai (`Grade`) wajib melekat pada `Enrollment` (bukan langsung ke `Student`) agar terikat pada pasangan mahasiswa–mata kuliah–periode yang sah.
8. Perhitungan IPS, IPK, dan transkrip dikelola oleh `TranscriptService` untuk menjaga enkapsulasi atribut `-ipk` pada `Student`.
9. Ketika `ProspectiveStudent` dinyatakan lulus seleksi PMB, sistem **membuat objek `Student` baru** melalui `StudentFactory` (bukan mengubah tipe class objek lama) dan mencatat keterkaitannya.
10. Mahasiswa hanya dapat melakukan presensi QR sesuai `Schedule` perkuliahan yang sedang berlangsung.

---

## 5. Penerapan Prinsip OOP & SOLID

- **Encapsulation:** Atribut sensitif seperti `-password` dan `-ipk` bersifat `private` dan hanya dapat diubah melalui method terkontrol.
- **Abstraction & Polymorphism:** `User` merupakan abstract class dengan method abstrak `tampilkanDashboard()` yang diimplementasikan berbeda oleh tiap peran.
- **Single Responsibility Principle (SRP):** Entitas hanya menyimpan data & perilaku inti; validasi KRS, kalkulasi transkrip, dan notifikasi dipisah ke service tersendiri.
- **Open/Closed Principle (OCP):** Penambahan aturan validasi KRS baru cukup menambah implementasi baru dari `IValidationRule` tanpa mengubah `KRSService`.
- **Dependency Inversion Principle (DIP):** `KRSService` bergantung pada abstraksi `IValidationRule` dan `INotificationService`, bukan implementasi konkret.
