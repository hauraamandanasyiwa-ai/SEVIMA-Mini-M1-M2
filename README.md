# SEVIMA-Mini — Desain Sistem Akademis dengan UML (Minggu 1–2)

Repositori ini berisi rancangan arsitektur sistem informasi akademik **SEVIMA-Mini** berbasis **Object-Oriented Programming (OOP)** dan prinsip **SOLID** menggunakan UML (Class Diagram, Use Case Diagram, dan Sequence Diagram) dengan pendekatan *AI-First Development*.

## Modul Sistem
1. **PMB (Penerimaan Mahasiswa Baru):** Pendaftaran, verifikasi berkas, hasil seleksi, dan pembuatan NIM otomatis.
2. **KRS (Kartu Rencana Studi):** Pengisian KRS dengan validasi prasyarat, batas maksimal 24 SKS, cegah *double enrollment*, cek bentrok jadwal, serta persetujuan Dosen PA.
3. **Jadwal & Ruangan:** Penjadwalan kelas, pengampu mata kuliah, dan ketersediaan ruangan.
4. **Nilai & Transkrip:** Input nilai oleh dosen, perhitungan IPS/IPK, dan generate transkrip PDF.
5. **Keuangan / SPP:** Pengelolaan tagihan dan status pembayaran SPP per periode akademik.
6. **Presensi QR Code:** Pencatatan kehadiran mahasiswa per sesi kuliah menggunakan QR Code.

---

## Struktur Repository

```text
SEVIMA_MINI_M1_M2/
├── diagrams/
│   ├── class-diagram.mmd                 # UML Class Diagram (SOLID + Strategy & Factory Pattern)
│   ├── usecase-diagram.mmd               # UML Use Case Diagram (<<include>> & <<extend>>)
│   ├── sequence-pmb.mmd                  # Sequence Diagram Proses PMB (+ alternate flow)
│   └── sequence-krs.mmd                  # Sequence Diagram Validasi KRS (+ error handling)
├── prompts/
│   ├── ai-usage-log.md                   # Log 6 iterasi prompt terbaik, revisi manual & OPAR
│   ├── prompt-class-diagram.md           # Evolusi prompt Class Diagram
│   └── prompt-use-case.md                # Prompt Use Case Diagram
├── docs/
│   ├── analisis-domain.md                # Analisis domain, entitas, relasi & aturan bisnis
│   ├── ADR-architecture-decisions.md     # Sumber Markdown ADR
│   ├── ADR-architecture-decisions.pdf    # Laporan Architecture Decision Record (PDF)
│   └── Final-UML-diagram.pdf             # Laporan Final Seluruh Diagram UML (PDF)
└── README.md
```

---

## Tautan Cepat Deliverables

| Deliverable | File | Ringkasan |
| :--- | :--- | :--- |
| **Class Diagram (35%)** | [diagrams/class-diagram.mmd](diagrams/class-diagram.mmd) | Hierarki `User` abstrak, 5 class inti + pendukung, `Composition`/`Aggregation`, `Strategy Pattern` & `Factory Pattern` |
| **Use Case Diagram (20%)** | [diagrams/usecase-diagram.mmd](diagrams/usecase-diagram.mmd) | 5 aktor (termasuk `Sistem PMB`), 6 modul, relasi `«include»` & `«extend»` |
| **Sequence Diagram PMB (12.5%)** | [diagrams/sequence-pmb.mmd](diagrams/sequence-pmb.mmd) | Alur pendaftaran, verifikasi berkas, *alternate flow* revisi/tidak lulus, & `StudentFactory` |
| **Sequence Diagram KRS (12.5%)** | [diagrams/sequence-krs.mmd](diagrams/sequence-krs.mmd) | Cek periode & SPP, validasi prasyarat/double/24 SKS/bentrok, status `Pending Approval`, notifikasi Dosen PA |
| **AI Usage Log (15%)** | [prompts/ai-usage-log.md](prompts/ai-usage-log.md) | 6 iterasi prompt terbaik, perbandingan AI vs revisi manual, alasan revisi, dan OPAR |
| **ADR PDF (5%)** | [docs/ADR-architecture-decisions.pdf](docs/ADR-architecture-decisions.pdf) | Keputusan arsitektur, *trade-off*, dan *design patterns* |
| **Final UML PDF** | [docs/Final-UML-diagram.pdf](docs/Final-UML-diagram.pdf) | Kompilasi laporan final seluruh diagram UML |

---

## Ringkasan Keputusan Desain Utama
- **Hierarki `User` (`<<abstract>>`):** Menarik atribut/method autentikasi bersama ke induk, menjaga `-password` tetap `private`, dan menerapkan polymorphism melalui `tampilkanDashboard()`.
- **Composition vs Aggregation:**
  - *Composition (`*--`):* `Enrollment`–`Grade`, `Course`–`Schedule`, `ProspectiveStudent`–`PMBRegistration`.
  - *Aggregation (`o--`):* `AcademicPeriod`–`Enrollment`, `Room`–`Schedule`, `KRSService`–`IValidationRule`.
- **Self-Association:** `Course` berelasi ke dirinya sendiri untuk memodelkan prasyarat mata kuliah.
- **SOLID & Design Patterns:**
  - **Strategy Pattern** (`KRSService` + `IValidationRule`) untuk validasi KRS modular (SRP & OCP).
  - **Factory Pattern** (`StudentFactory`) untuk transisi `ProspectiveStudent` lulus menjadi objek `Student` baru + generate NIM otomatis.
  - **Dependency Inversion** (`INotificationService`) & pemisahan `TranscriptService` agar seluruh class ramping dan bebas *God Object*.

---

## Cara Melihat Diagram
- Buka langsung file `.mmd` di GitHub (GitHub merender Mermaid secara otomatis), atau
- Salin isi file `.mmd` ke [Mermaid Live Editor](https://mermaid.live), atau
- Buka laporan kompilasi di [docs/Final-UML-diagram.pdf](docs/Final-UML-diagram.pdf).
