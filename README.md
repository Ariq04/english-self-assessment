# 🎓 English Self-Assessment Quiz Platform

[![Python Version](https://img.shields.io/badge/python-3.11%20%7C%203.12-blue.svg)](https://www.python.org/)
[![Framework](https://img.shields.io/badge/framework-Streamlit%201.35+-red.svg)](https://streamlit.io/)
[![Database](https://img.shields.io/badge/database-SQLite%20(WAL%20Mode)-lightgrey.svg)](https://www.sqlite.org/)
[![Tests](https://img.shields.io/badge/tests-132%20passed-success.svg)](https://docs.pytest.org/)
[![Deployment](https://img.shields.io/badge/deploy-Railway-purple.svg)](https://railway.app/)

Platform asesmen mandiri Bahasa Inggris berbasis web yang interaktif, modular, dan didukung analisis performa butir soal. Dirancang untuk mengevaluasi kompetensi peserta secara terstruktur pada tiga pilar utama: **Grammar**, **Reading Comprehension**, dan **Listening Comprehension**.

---

## 🌟 Fitur Utama Aplikasi

### 1. Alur Asesmen Peserta (*Participant Workflow*)
- **Validasi Identitas & Session**: Registrasi peserta dengan validasi format email (RFC/DNS-compliant) dan fitur *lookup* riwayat otomatis bagi peserta terdaftar.
- **Dua Moda Asesmen**:
  - **Standard Difficulty Mode**: Pemilihan bebas tingkat kesulitan (*Easy*, *Medium*, *Hard*) per kategori.
  - **Skill Progression Mode**: Pendekatan kurikulum berjenjang berbasis skill TOEFL (Skill 1 s/d N) dengan ambang kelulusan (*passing score*) serta **Final Exam (Comprehensive Sampling)**.
- **Quiz Engine Interaktif**:
  - Algoritma pemilihan soal acak dengan jaminan tanpa pengulangan (*zero-repetition guarantee*) dalam satu sesi.
  - Pengacakan posisi opsi jawaban (*distractor shuffle*) dengan umpan balik visual dan pembahasan instan (*instant explanation*).
  - Modul Reading terintegrasi teks bacaan majemuk (*multi-question passages*).
  - Modul Listening dengan pemutar audio HTML5 dan transkrip percakapan.
- **Riwayat & Evaluasi Personal**: Rekapitulasi skor longitudinal, tren perkembangan akurasi, dan tinjauan detail setiap jawaban peserta.

### 2. Panel Administrasi & Analisis Data
- **Keamanan Kredensial**: Autentikasi administrator menggunakan salted `bcrypt` (12 rounds) dengan proteksi rute (`require_admin_auth`).
- **Dashboard Eksekutif**: Metrik KPI ringkasan platform (Total Users, Attempts, Questions, Active Questions) dengan visualisasi interaktif Plotly.
- **Manajemen Bank Soal (CRUD)**: Pengelolaan penuh butir soal, pengaturan status (*Draft*, *Active*, *Inactive*, *Archived*), serta aksi massal (*bulk toggles* & *bulk delete*).
- **Pipeline Impor Soal (DOCX & Excel)**:
  - Ekstraksi otomatis butir soal dari dokumen Word (`.docx`) dan spreadsheet (`.xlsx`).
  - Validasi keamanan arsip: verifikasi *magic-bytes* ZIP, pembatasan ukuran file, dan mitigasi *decompression bomb*.
  - Ekspor seluruh bank soal ke format Excel (`.xlsx`).
- **Analitik Psikometrik Butir Soal**:
  - Evaluasi akurasi empiris peserta per butir soal vs tingkat kesulitan teoretis.
  - Deteksi anomali butir soal (*needs review flag*) untuk kalibrasi kualitas bank soal.
- **Pengaturan Kredensial Mandiri**: Fitur pembaruan username dan kata sandi admin secara aman via portal settings.

---

## 🏗️ Arsitektur Perangkat Lunak

Aplikasi ini menerapkan **Layered Architecture (Separation of Concerns)** untuk memisahkan antarmuka pengguna, logika bisnis, dan akses data:

```text
├── Presentation Layer   : Streamlit multi-page routing dinamis (st.navigation)
├── Business Logic Layer : Services terisolasi (Scoring, Selector, Analytics, Auth)
├── Data Access Layer    : Repository Pattern (100% Parameterized SQL Queries)
└── Storage Layer        : SQLite terindeks dengan WAL (Write-Ahead Logging) Mode
```

### Struktur Direktori Proyek

```text
english-self-assessment/
├── .streamlit/
│   ├── config.toml                     # Konfigurasi server, tema & UI Streamlit
│   └── secrets.toml.example            # Template rahasia lingkungan (bukan file rahasia asli)
├── app.py                              # Entry point aplikasi & router navigasi dinamis
├── Procfile                            # Start command process web untuk Railway
├── railway.json                        # Konfigurasi build Nixpacks & restart policy Railway
├── runtime.txt                         # Pinning versi runtime (python-3.11.9)
├── requirements.txt                    # Dependensi produksi minimal & terverifikasi
├── README.md                           # Dokumentasi komprehensif & panduan arsitektur
├── PRD.md                              # Product Requirement Document
│
├── config/                             # Konfigurasi terpusat
│   ├── constants.py                    # Enum kategori, difficulty, CEFR level
│   └── settings.py                     # Resolusi path dinamis (DATA_DIR, DATABASE_PATH)
│
├── database/                           # Layer persistensi data
│   ├── db.py                           # Koneksi SQLite, WAL mode & DDL initialization
│   ├── schema.py                       # DDL 13 tabel, 11 indeks, foreign keys, constraints
│   ├── models.py                       # Python Dataclass entities (User, Admin, Question, etc.)
│   ├── queries.py                      # 100% Parameterized SQL query repository
│   └── seed_data.py                    # Seeding otomatis bank soal awal
│
├── services/                           # Pure Business Logic Layer
│   ├── user_service.py                 # Registrasi & manajemen session peserta
│   ├── quiz_engine.py                  # State machine kuis, tracking waktu & attempt
│   ├── question_selector.py            # Algoritma seleksi soal acak & anti-repetisi
│   ├── scoring_service.py              # Kalkulasi skor persentase & pemetaan kecakapan
│   ├── history_service.py              # Agregasi riwayat & grafik perkembangan peserta
│   ├── admin_service.py                # Autentikasi bcrypt admin & proteksi rute
│   ├── admin_dashboard_service.py      # Agregasi metrik KPI & visualisasi Plotly
│   ├── question_service.py             # Validasi payload & CRUD bank soal
│   ├── import_service.py               # Batch executor impor Word & Excel
│   └── analytics_service.py            # Analitik performa & akurasi empiris butir soal
│
├── components/                         # UI Reusable Components
│   ├── navigation.py                   # Custom header bar & sidebar admin
│   └── ui_styles.py                    # Injeksi CSS global & card styling
│
├── pages/                              # Presentation Layer (14 Halaman Terdaftar)
│   ├── 01_Home.py                      # Landing page & pengenalan platform
│   ├── 02_Identity.py                  # Registrasi identitas & verifikasi peserta
│   ├── 03_Test_Selection.py            # Seleksi kategori, level, & TOEFL skill progression
│   ├── 04_Quiz.py                      # Active quiz engine interface & instant feedback
│   ├── 05_Result.py                    # Skor, breakdown performa & kunci pembahasan
│   ├── 06_History.py                   # Riwayat longitudinal & grafik tren pengerjaan
│   ├── 90_Admin_Login.py               # Portal autentikasi administrator
│   ├── 91_Admin_Dashboard.py           # Dashboard metrik platform & grafik Plotly
│   ├── 92_Admin_Users.py               # Manajemen direktori peserta
│   ├── 93_Admin_Question_Bank.py       # CRUD & bulk actions bank soal
│   ├── 94_Admin_Import.py              # Pipeline impor soal DOCX & Excel
│   ├── 95_Admin_Analytics.py           # Analitik performa & kalibrasi kesulitan soal
│   ├── 96_Admin_Results.py             # Ledger log pengerjaan seluruh peserta
│   └── 97_Admin_Settings.py            # Manajemen kredensial & kata sandi admin
│
├── utils/                              # Standalone Utilities & Parsers
│   ├── admin_init.py                   # CLI provisioning akun superadmin aman
│   ├── audio.py                        # Audio player & safe playback handlers
│   ├── backup_db.py                    # Hot atomic SQLite backup utility
│   ├── difficulty.py                   # Rule-based difficulty classifier
│   ├── docx_parser.py                  # Resilient Word (.docx) document parser
│   ├── excel_parser.py                 # Excel (.xlsx) importer & exporter
│   ├── security.py                     # Sanitasi filename, magic-bytes, mitigasi zip-bomb
│   └── validators.py                   # Validasi email (DNS/RFC) & nama
│
├── data/                               # Direktori data lokal & template
│   ├── contoh_soal_berbasis_skill.xlsx # Contoh soal referensi
│   └── template_soal_berbasis_skill.xlsx # Template impor bank soal
│
├── audio/                              # Direktori media audio
│   ├── feedback/                       # Sound effects evaluasi jawaban
│   └── listening/                      # Audio prompts listening comprehension
│
└── tests/                              # 20 File Automated Test Suites (132 Tests)
```

---

## 💾 Desain Basis Data & Konkurensi

- **Relational Schema**: 13 tabel terstruktur, 11 indeks performa, penegakan integritas referensial (`PRAGMA foreign_keys = ON;` dengan `CASCADE` dan `RESTRICT`).
- **High Concurrency Tuning**:
  - `PRAGMA journal_mode = WAL;` (Write-Ahead Logging: pembaca tidak memblokir penulis).
  - `PRAGMA synchronous = NORMAL;` (Efisiensi I/O disk).
  - `timeout = 30.0` detik pada koneksi untuk mencegah isu *database locked*.

---

## 🚀 Panduan Menjalankan Aplikasi Secara Lokal

### 1. Kebutuhan Sistem
- Python 3.10, 3.11, atau 3.12
- Git

### 2. Instalasi Lokal
```bash
# Clone repository
git clone https://github.com/Ariq04/english-self-assessment.git
cd english-self-assessment

# Buat virtual environment
python -m venv .venv

# Aktivasi environment (Windows PowerShell)
.venv\Scripts\Activate.ps1
# Aktivasi environment (Linux / macOS)
source .venv/bin/activate

# Install dependensi
pip install -r requirements.txt
```

### 3. Inisialisasi Akun Administrator
Gunakan utilitas CLI bawaan untuk membuat akun administrator awal secara aman:
```bash
python utils/admin_init.py
```

### 4. Menjalankan Server
```bash
streamlit run app.py
```
Akses antarmuka aplikasi melalui peramban di `http://localhost:8501`.

---

## 🧪 Pengujian Otomatis (*Automated Testing*)

Proyek ini diverifikasi menggunakan rangkaian unit & integration tests menyeluruh:
```bash
python -m pytest -v
```
**Hasil Verifikasi Aktual:**
```text
============================ 132 passed in 20.50s =============================
```
Mencakup 20 modul pengujian: autentikasi admin, parsing dokumen DOCX/Excel, algoritma pengacakan soal tanpa repetisi, kalkulasi skor, audio handling, integritas database, dan pengerasan keamanan (*security hardening*).

---

## 🌐 Catatan Deployment Produksi (Railway)

Aplikasi telah dikonfigurasi untuk cloud deployment di **Railway**:
- **Build Engine**: `railway.json` mengonfigurasi builder `NIXPACKS` dengan versi Python `3.11.9` (`runtime.txt`).
- **Process Manager**: `Procfile` menjalankan Streamlit dengan pengikatan port dinamis `${PORT:-8501}`.
- **Catatan Penyimpanan Cloud**:
  - Secara default, container PaaS bersifat *ephemeral*. Untuk mempertahankan database dan riwayat ujian lintas *redeploy*, Railway memerlukan pemasangan **Persistent Volume** yang diarahkan ke `/app/data` (dikonfigurasi via variabel `DATA_DIR=/app/data`).
  - File audio statis disertakan dalam direktori aplikasi (`audio/`).
