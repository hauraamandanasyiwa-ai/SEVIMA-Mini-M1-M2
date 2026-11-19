# PROMPT ENGINEERING — CLASS DIAGRAM SEVIMA-MINI

Catatan: Log lengkap seluruh 6 iterasi prompt (termasuk perbandingan output AI vs revisi manual dan OPAR sesuai Charter) terdokumentasi di [`ai-usage-log.md`](./ai-usage-log.md).

## Iterasi 1 — Draft Class Diagram Awal

```text
Saya sedang mengembangkan sistem informasi akademik sederhana bernama SEVIMA Mini menggunakan konsep Object Oriented Programming (OOP).

Sistem memiliki enam modul utama:
1. Penerimaan Mahasiswa Baru (PMB).
2. Kartu Rencana Studi (KRS).
3. Jadwal dan Ruangan.
4. Nilai dan Transkrip.
5. Keuangan atau Pembayaran SPP.
6. Presensi menggunakan QR Code.

Berdasarkan dokumen analisis domain, buatlah UML Class Diagram yang mencakup seluruh class, atribut, method, dan hubungan antarclass.

Gunakan ketentuan berikut:
- User merupakan abstract class.
- Student, Lecturer, Admin, dan ProspectiveStudent mewarisi User.
- Enrollment menjadi class penghubung antara Student dan Course.
- Grade harus terhubung dengan Enrollment.
- Setiap class memiliki tanggung jawab yang jelas.
- Terapkan prinsip Single Responsibility Principle (SRP).
- Gunakan visibility public (+), private (-), dan protected (#).
- Tampilkan multiplicity pada setiap hubungan antarclass.
- Hindari pembuatan class yang memiliki terlalu banyak tanggung jawab.

Hasilkan diagram menggunakan sintaks Mermaid Class Diagram yang valid dan dapat ditampilkan di Mermaid Live Editor.
```

## Iterasi 2 — Penyempurnaan Relasi UML & Audit SOLID (Final Prompt)

```text
# Role
Anda adalah senior software architect berpengalaman dalam desain OOP dan prinsip SOLID.

# Konteks
Sistem informasi akademik SEVIMA-Mini dengan 6 modul: PMB, KRS, Jadwal & Ruangan, Nilai & Transkrip, Keuangan/SPP, dan Presensi QR Code.

# Komponen
- Hierarki User (<<abstract>> User -> ProspectiveStudent, Student, Lecturer, Admin) dengan polymorphism method tampilkanDashboard().
- Modul PMB: PMBRegistration dan StudentFactory (Factory Pattern untuk transisi ProspectiveStudent lulus -> Student baru + generate NIM).
- Class inti: AcademicPeriod, Course (self-association prasyarat), Enrollment, Grade, Schedule, Room, Payment, Attendance.
- Service & Strategy Pattern: KRSService, IValidationRule (MaxSKSValidation, PrerequisiteValidation, ScheduleConflictValidation, DoubleEnrollmentValidation), TranscriptService, INotificationService, NotificationService.

# Constraint
- Terapkan SOLID (SRP, OCP, DIP), tidak ada God Object, dan tidak ada circular dependency.
- Gunakan Composition (*--) untuk relasi kuat (Enrollment-Grade, Course-Schedule, ProspectiveStudent-PMBRegistration).
- Gunakan Aggregation (o--) untuk relasi lemah (AcademicPeriod-Enrollment, Room-Schedule, KRSService-IValidationRule).
- Pastikan tidak ada class yatim (hubungkan Lecturer ke Schedule & Enrollment, Admin ke PMBRegistration, Attendance ke Schedule).
- Wajib tampilkan visibility modifier (+, -, #) dan kardinalitas di setiap relasi.

# Format Output
Keluarkan hanya kode Mermaid classDiagram yang valid (gunakan sintaks ~ untuk generic seperti List~Schedule~).
```
