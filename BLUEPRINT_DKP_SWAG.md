# 🏗️ Arsitektur & Scaffolding Sistem — DKP SWAG

> **The Blueprint**: Use Case, Sequence Diagram, Domain Model
> **Project**: Aplikasi Pendataan Makam & Krematorium — DKP Kota Surabaya

---

## 📑 Daftar Isi

1. [Use Case Diagram](#1-use-case-diagram)
2. [Sequence Diagram](#2-sequence-diagram)
3. [Domain Model](#3-domain-model)

---

## 1. Use Case Diagram

### 1.1 Use Case Utama (High-Level)

```mermaid
graph TB
    subgraph "🧑‍💼 Aktor"
        ADMIN["👤 Administrator"]
        PETUGAS["👤 Petugas Lapangan"]
        PEJABAT["👤 Pejabat / Kadis"]
        PELAPOR["👤 Pelapor / Ahli Waris"]
    end

    subgraph "📦 DKP SWAG System"
        UC1["🔐 Login / Logout"]
        UC2["📋 Kelola Data Almarhum"]
        UC3["🪦 Kelola Makam & Plot"]
        UC4["🔥 Kelola Krematorium"]
        UC5["📸 Input Kegiatan Harian (Giat)"]
        UC6["💰 Kelola Tagihan & Retribusi"]
        UC7["💳 Proses Pembayaran"]
        UC8["📄 Cetak Surat & Laporan"]
        UC9["🔍 Pencarian Data"]
        UC10["👥 Kelola Master Data"]
        UC11["📰 Kelola Berita"]
        UC12["📜 Lihat Log Aktivitas"]
        UC13["✍️ Tanda Tangan Elektronik"]
    end

    ADMIN --> UC1
    ADMIN --> UC2
    ADMIN --> UC3
    ADMIN --> UC4
    ADMIN --> UC5
    ADMIN --> UC6
    ADMIN --> UC7
    ADMIN --> UC8
    ADMIN --> UC9
    ADMIN --> UC10
    ADMIN --> UC11
    ADMIN --> UC12
    ADMIN --> UC13

    PETUGAS --> UC1
    PETUGAS --> UC2
    PETUGAS --> UC5
    PETUGAS --> UC9

    PEJABAT --> UC1
    PEJABAT --> UC8
    PEJABAT --> UC13
    PEJABAT --> UC9

    PELAPOR --> UC2
    PELAPOR --> UC7
```

### 1.2 Use Case Detail — Kelola Almarhum

```mermaid
graph TB
    subgraph "👤 Aktor"
        PETUGAS2["Petugas"]
        ADMIN2["Admin"]
    end

    subgraph "📋 Almarhum Management"
        UC_CREATE["➕ Tambah Almarhum"]
        UC_READ["👁️ Lihat Detail"]
        UC_UPDATE["✏️ Edit Data"]
        UC_DELETE["🗑️ Hapus Data"]
        UC_UPLOAD["📎 Upload Dokumen\n(KTP, SKTM, Foto Kuburan)"]
        UC_AMBIL["📤 Catat Pengambilan\nJenazah"]
        UC_KOMPENSASI["💵 Entri Kompensasi"]
    end

    subgraph "🔗 Include"
        UC_VALIDASI["✅ Validasi NIK"]
        UC_CEK_PLOT["🪦 Cek Ketersediaan Plot"]
        UC_LOG["📜 Catat Log Aktivitas"]
    end

    ADMIN2 --> UC_CREATE
    ADMIN2 --> UC_READ
    ADMIN2 --> UC_UPDATE
    ADMIN2 --> UC_DELETE
    ADMIN2 --> UC_UPLOAD
    ADMIN2 --> UC_AMBIL
    ADMIN2 --> UC_KOMPENSASI

    PETUGAS2 --> UC_CREATE
    PETUGAS2 --> UC_READ
    PETUGAS2 --> UC_UPLOAD
    PETUGAS2 --> UC_AMBIL

    UC_CREATE -.-> UC_VALIDASI
    UC_CREATE -.-> UC_CEK_PLOT
    UC_CREATE -.-> UC_LOG
    UC_UPDATE -.-> UC_LOG
    UC_DELETE -.-> UC_LOG
    UC_AMBIL -.-> UC_LOG
```

### 1.3 Use Case Detail — Tagihan & Pembayaran

```mermaid
graph LR
    subgraph "👤 Aktor"
        ADMIN3["Admin"]
        PELAPOR2["Ahli Waris"]
    end

    subgraph "💰 Tagihan & Pembayaran"
        UC_GEN["📝 Generate Tagihan"]
        UC_VIEW_TAGIHAN["📋 Lihat Tagihan"]
        UC_BAYAR["💵 Proses Pembayaran"]
        UC_LUNAS["✅ Konfirmasi Lunas"]
        UC_SKTM["🆓 Bebas Tagihan (SKTM)"]
        UC_CETAK_TAGIHAN["🖨️ Cetak Surat Tagihan"]
        UC_CETAK_LUNAS["🖨️ Cetak Bukti Lunas"]
        UC_DENDA["⚠️ Hitung Denda"]
    end

    ADMIN3 --> UC_GEN
    ADMIN3 --> UC_VIEW_TAGIHAN
    ADMIN3 --> UC_BAYAR
    ADMIN3 --> UC_LUNAS
    ADMIN3 --> UC_SKTM
    ADMIN3 --> UC_CETAK_TAGIHAN
    ADMIN3 --> UC_CETAK_LUNAS
    ADMIN3 --> UC_DENDA

    PELAPOR2 --> UC_VIEW_TAGIHAN
    PELAPOR2 --> UC_BAYAR

    UC_LUNAS -.-> UC_CETAK_LUNAS
    UC_SKTM -.-> UC_GEN
```

---

## 2. Sequence Diagram

### 2.1 Registrasi Almarhum Baru (Happy Path)

```mermaid
sequenceDiagram
    actor P as Petugas
    participant V as View Form Almarhum
    participant C as Almarhum Controller
    participant M as Model Almarhum
    participant D as Database

    P->>V: Buka form tambah almarhum
    V->>C: GET /almarhum_ext/tambah
    C->>M: get_kecamatan() + get_kelurahan()
    M->>D: SELECT FROM kecamatan dan kelurahan
    D-->>M: data wilayah
    M-->>C: result()
    C-->>V: Render form dengan dropdown wilayah

    P->>V: Isi form NIK, Nama, Tgl Lahir/Mati, Agama, Plot Makam
    P->>V: Upload KTP, SKTM opsional
    V->>C: POST /almarhum_ext/simpan

    C->>C: Validasi form
    alt Validasi Gagal
        C-->>V: Error message, kembali ke form
    else Validasi Sukses
        C->>M: cek_nik(NIK_ALMARHUM)
        M->>D: SELECT COUNT FROM almarhum WHERE NIK_ALMARHUM
        D-->>M: 0 NIK belum terdaftar
        M-->>C: available
        C->>M: insert_almarhum(data)
        M->>D: INSERT INTO almarhum
        D-->>M: success
        C->>M: insert_ahli_waris(data_waris)
        M->>D: INSERT INTO ahli_waris
        C->>M: update_isi_makam(ID_ISI, status)
        M->>D: UPDATE isi_makam SET terisi = 1
        C->>M: insert_log(user, action, table)
        M->>D: INSERT INTO log_activity
        M-->>C: success
        C-->>V: Redirect dengan flash message Data berhasil disimpan
        V-->>P: Tampilkan notifikasi sukses
    end
```

### 2.2 Proses Pembayaran & Pelunasan

```mermaid
sequenceDiagram
    actor AW as Ahli Waris
    actor AD as Admin
    participant V as View Tagihan
    participant C as Tagihan Controller
    participant MT as Model_Tagihan
    participant MP as Model_Pembayaran
    participant D as Database

    Note over AW,D: === GENERATE TAGIHAN (Admin) ===
    AD->>C: Generate tagihan untuk NIK almarhum
    C->>MT: generate_tagihan(NIK_ALMARHUM)
    MT->>D: SELECT * FROM almarhum WHERE NIK_ALMARHUM = ?
    D-->>MT: data almarhum + retribusi
    MT->>MT: Hitung tarif retribusi + denda (jika ada)
    MT->>D: INSERT INTO tagihan (...) VALUES (...)
    D-->>MT: tagihan_id
    MT-->>C: Data tagihan

    Note over AW,D: === CEK TAGIHAN (Ahli Waris) ===
    AW->>C: Lihat status tagihan
    C->>MT: get_tagihan_by_nik(NIK)
    MT->>D: SELECT * FROM tagihan WHERE NIK_ALMARHUM = ?
    D-->>MT: list tagihan
    MT-->>C: result()
    C-->>V: Tampilkan daftar tagihan

    Note over AD,D: === PROSES PEMBAYARAN (Admin) ===
    AW->>AD: Melakukan pembayaran
    AD->>C: Input pembayaran (jumlah, tgl, metode)
    C->>MP: insert_pembayaran(ID_TAGIHAN, data)
    MP->>D: INSERT INTO pembayaran (...) VALUES (...)

    alt Lunas
        C->>MT: update_status_lunas(ID_TAGIHAN)
        MT->>D: UPDATE tagihan SET STATUS = 'LUNAS'
        C->>MP: generate_bukti_lunas(ID_TAGIHAN)
        MP-->>C: bukti lunas
        C-->>V: Tampilkan + Cetak Bukti Lunas
        V-->>AW: Bukti lunas
    else Belum Lunas / Cicilan
        C->>MT: update_sisa_tagihan(ID_TAGIHAN, sisa)
        MT->>D: UPDATE tagihan SET SISA = ?
        MT-->>C: success
        C-->>V: Notifikasi sisa tagihan
    end
```

### 2.3 Cetak Laporan Pendapatan

```mermaid
sequenceDiagram
    actor P as Pejabat / Admin
    participant V as View Laporan
    participant C as Laporan Controller
    participant ML as Model_GiatLaporan
    participant D as Database
    participant PDF as PDF Generator

    P->>V: Pilih periode & jenis laporan
    V->>C: POST /laporan/pendapatan (tgl_mulai, tgl_akhir)

    C->>ML: get_pendapatan_range(tgl_mulai, tgl_akhir)
    ML->>D: SELECT p.*, a.NAMA_ALMARHUM, m.NAMA_MAKAM<br/>FROM pembayaran p<br/>JOIN tagihan t ON p.ID_TAGIHAN = t.ID_TAGIHAN<br/>JOIN almarhum a ON t.NIK_ALMARHUM = a.NIK_ALMARHUM<br/>WHERE p.TGL_BAYAR BETWEEN ? AND ?
    D-->>ML: result set
    ML-->>C: data laporan

    alt Tipe: Tampil di Layar
        C-->>V: Render tabel laporan dgn total
        V-->>P: Lihat laporan di browser
    else Tipe: Cetak PDF
        C->>PDF: generate_pdf(data, periode, pejabat_ttd)
        PDF->>D: SELECT * FROM pejabat_ttd WHERE JABATAN = ?
        D-->>PDF: nama & nip pejabat
        PDF->>PDF: Render PDF (kop surat, tabel, ttd)
        PDF-->>C: file PDF
        C-->>V: Download / Stream PDF
        V-->>P: File laporan pendapatan.pdf
    end
```

### 2.4 Autentikasi & Session

```mermaid
sequenceDiagram
    actor U as User
    participant V as Login Form
    participant C as Home Controller
    participant Auth as Auth Library
    participant MP as Model_Pengguna
    participant D as Database

    U->>V: Akses halaman aplikasi
    V->>C: GET /home
    C->>Auth: is_logged_in()
    Auth-->>C: FALSE
    C-->>V: Tampilkan form login

    U->>V: Input username & password
    V->>C: POST /home/login

    C->>C: form_validation (required, trim)

    alt Validasi Form Gagal
        C-->>V: Error: "Username & Password wajib diisi"
    else Validasi Sukses
        C->>Auth: login(username, password)
        Auth->>MP: cek_login(username, password)
        MP->>D: SELECT * FROM pengguna<br/>WHERE USERNAME = ? AND PASSWORD = MD5(?)
        D-->>MP: user data (atau empty)

        alt User Tidak Ditemukan
            MP-->>Auth: FALSE
            Auth-->>C: Login gagal
            C-->>V: "Username atau Password salah!"
        else User Ditemukan
            MP-->>Auth: user_data (ID, nama, role)
            Auth->>Auth: set_session(user_data)
            Auth->>MP: insert_log(ID_PENGGUNA, 'LOGIN')
            MP->>D: INSERT INTO log_activity (USER, ACTION, ...)
            Auth-->>C: Login sukses
            C-->>V: Redirect ke dashboard / pencarian
            V-->>U: Halaman Dashboard
        end
    end
```

---

## 3. Domain Model

### 3.1 Entity Relationship Diagram (ERD)

```mermaid
erDiagram
    kecamatan ||--o{ kelurahan : "memiliki"
    kelurahan ||--o{ makam : "wilayah"
  
    makam ||--o{ isi_makam : "terdiri dari"
    makam ||--o{ kapasitas_blok : "memiliki blok"
  
    isi_makam ||--o{ almarhum : "ditempati"
    ahli_waris ||--o{ almarhum : "mewakili"
    pelapor ||--o{ almarhum : "melaporkan"
  
    almarhum ||--o{ tagihan : "dikenakan"
    tagihan ||--o{ pembayaran : "dilunasi dengan"
    tagihan ||--o{ tagihanorang : "detail per orang"
  
    almarhum ||--o{ kompensasi : "mendapat"
  
    krematorium ||--o{ almarhum_kremasi : "melayani"
    ahli_waris ||--o{ almarhum_kremasi : "mewakili"
    pelapor ||--o{ almarhum_kremasi : "melaporkan"
  
    pengguna ||--o{ log_activity : "melakukan"
    petugas ||--o{ giat_makam : "mencatat"
    petugas ||--o{ giat_krematorium : "mencatat"
  
    giat_makam ||--o{ foto_giat_makam : "didokumentasikan"
    giat_makam }o--|| giat_target : "memenuhi"
    giat_makam }o--|| giat_tarifmakam : "menggunakan tarif"
  
    giat_krematorium }o--|| giat_target : "memenuhi"
    giat_krematorium }o--|| giat_tarifkrematorium : "menggunakan tarif"
  
    pejabat ||--o{ pejabat_ttd : "ditunjuk sebagai"
  
    bulan ||--o{ giat_makam : "periode"
    bulan ||--o{ giat_krematorium : "periode"
  
    sarana ||--o{ makam : "fasilitas"
  
    berita }o--|| pengguna : "dipublikasikan oleh"

    %% --- ENTITY ATTRIBUTES ---

    almarhum {
        varchar NIK_ALMARHUM PK
        int ID_ISI FK
        varchar NIK_WARIS FK
        varchar NAMA_ALMARHUM
        varchar ALAMAT_ALMARHUM
        varchar AGAMA
        varchar KELAMIN
        date TGL_LAHIR
        date TGL_MATI
        date TGL_MAKAM
        int UMUR
        int BEBAS_TAGIHAN "1=Kena, 0=Bebas"
        varchar FILE_SKTM
        varchar FOTO_KUBURAN
        varchar KTP_WARIS
        int ID_PELAPOR FK
        varchar NIK_PENGAMBIL
        varchar NAMA_PENGAMBIL
        varchar ALAMAT_PENGAMBIL
        date TGL_AMBIL
        int RETRIBUSI
        varchar TEMBUSAN
    }

    ahli_waris {
        varchar NIK_WARIS PK
        varchar NAMA_WARIS
        varchar ALAMAT_WARIS
        varchar TELP_WARIS
    }

    pelapor {
        int ID_PELAPOR PK
        varchar NAMA_PELAPOR
        varchar ALAMAT_PELAPOR
        varchar TELP_PELAPOR
        varchar KTP_PELAPOR
    }

    makam {
        int ID_MAKAM PK
        varchar NAMA_MAKAM
        int ID_KELURAHAN FK
        varchar ALAMAT
        int KAPASITAS
        int TERISI
    }

    isi_makam {
        int ID_ISI PK
        int ID_MAKAM FK
        varchar BLOK
        varchar NOMOR
        int TERISI "0=Kosong, 1=Terisi"
    }

    kapasitas_blok {
        int ID_BLOK PK
        int ID_MAKAM FK
        varchar NAMA_BLOK
        int KAPASITAS
        int TERISI
    }

    krematorium {
        int KODE_KREMATORIUM PK
        varchar NAMA_KREMATORIUM
        varchar ALAMAT
    }

    almarhum_kremasi {
        varchar NIK_ALMARHUM PK
        varchar NIK_WARIS FK
        varchar NAMA_ALMARHUM
        varchar ALAMAT_ALMARHUM
        varchar AGAMA
        varchar KELAMIN
        date TGL_LAHIR
        date TGL_MATI
        datetime TGL_KREMASI
        int KODE_KREMATORIUM FK
        int MODEL_PETI
        int SEWA_TEMPAT
        int UMUR
        int BEBAS_TAGIHAN
        varchar FILE_SKTM
        varchar KTP_WARIS
        int ID_PELAPOR FK
        varchar NIK_PENGAMBIL
        varchar NAMA_PENGAMBIL
        date TGL_AMBIL
        int RETRIBUSI
    }

    tagihan {
        int ID_TAGIHAN PK
        varchar NIK_ALMARHUM FK
        varchar NO_TAGIHAN
        date TGL_TAGIHAN
        int JUMLAH
        int DENDA
        int TOTAL
        int SISA
        varchar STATUS "LUNAS/BELUM"
    }

    pembayaran {
        int ID_PEMBAYARAN PK
        int ID_TAGIHAN FK
        date TGL_BAYAR
        int JUMLAH
        varchar METODE
        varchar BUKTI
    }

    tagihanorang {
        int ID_TAGIHAN FK
        varchar NAMA
        int JUMLAH
        varchar KETERANGAN
    }

    kompensasi {
        int ID_KOMPENSASI PK
        varchar NIK_ALMARHUM FK
        date TGL_KOMPENSASI
        int JUMLAH
        varchar KETERANGAN
    }

    giat_makam {
        bigint id_gm PK
        date tanggal
        varchar blok
        int petak
        int bongkar
        int sewa
        int retribusi
        int kompensasi
        varchar foto
        varchar ket
        int ID_PETUGAS FK
    }

    giat_krematorium {
        bigint id_km PK
        date tanggal
        int id_krematorium FK
        int mp "Model Peti"
        int t2 "Tempat 2 jam"
        int t3 "Tempat 3 jam"
        int t6 "Tempat 6 jam"
        int ret "Retribusi"
        int sp "Sewa Peti"
        int sal "Salep"
        int pr "Perawatan"
        int rat "Ratus"
        int ag "Angsana"
        int sr "Serai"
        varchar foto
        varchar ket
    }

    pengguna {
        int ID_PENGGUNA PK
        varchar USERNAME
        varchar PASSWORD
        varchar NAMA_LENGKAP
        varchar ROLE "admin/petugas/pejabat"
        int AKTIF
    }

    petugas {
        int ID_PETUGAS PK
        varchar NAMA_PETUGAS
        varchar NIP
        varchar JABATAN
        varchar TELP
    }

    pejabat {
        int ID_PEJABAT PK
        varchar NAMA_PEJABAT
        varchar NIP
        varchar JABATAN
    }

    pejabat_ttd {
        int ID_TTD PK
        int ID_PEJABAT FK
        varchar JABATAN_TTD
        varchar KETERANGAN
    }

    log_activity {
        bigint ID_LOG PK
        int ID_PENGGUNA FK
        varchar ACTION
        varchar TABLE_NAME
        int RECORD_ID
        text KETERANGAN
        datetime WAKTU
    }

    berita {
        int id_berita PK
        varchar judul
        text keterangan
        varchar gambar
        date date
    }

    kecamatan {
        int ID_KECAMATAN PK
        varchar NAMA_KECAMATAN
    }

    kelurahan {
        int ID_KELURAHAN PK
        int ID_KECAMATAN FK
        varchar NAMA_KELURAHAN
    }

    sarana {
        int ID_SARANA PK
        varchar NAMA_SARANA
        int ID_MAKAM FK
        varchar KONDISI
        int JUMLAH
    }

    bulan {
        int ID_BULAN PK
        varchar BULAN
    }

    giat_target {
        int id_target PK
        int tahun
        int target_makam
        int target_kremasi
    }

    giat_tarifmakam {
        int id_tarif PK
        int tahun
        int tarif_petak
        int tarif_bongkar
        int tarif_sewa
        int tarif_retribusi
        int tarif_kompensasi
    }

    giat_tarifkrematorium {
        int id_tarif PK
        int tahun
        int tarif_mp
        int tarif_t2
        int tarif_t3
        int tarif_t6
        int tarif_ret
        int tarif_sp
        int tarif_sal
        int tarif_pr
        int tarif_rat
        int tarif_ag
        int tarif_sr
    }

    foto_giat_makam {
        bigint id_fgm PK
        bigint id_gm FK
        varchar namafile
        varchar foto
    }

    temp_tbl {
        varchar key1
        varchar key2
        varchar value
    }
```

### 3.2 Class Diagram — MVC Architecture

```mermaid
classDiagram
    direction TB

    %% --- BASE ---
    class CI_Controller {
        +load model, view, library
        +input post/get
        +session
    }

    class CI_Model {
        +db query builder
        +getAll(table)
        +insert(table, data)
        +update(table, data, where)
        +delete(table, where)
    }

    %% --- CONTROLLERS ---
    class Home {
        +index()
        +login()
        +logout()
    }

    class AlmarhumExt {
        +index()
        +tambah()
        +simpan()
        +edit(NIK)
        +update()
        +hapus(NIK)
        +detail(NIK)
        +pengambilan(NIK)
        +cetak_pdf()
    }

    class GiatMakam {
        +index()
        +tambah()
        +simpan()
        +edit(id)
        +update()
        +hapus(id)
        +upload_foto()
    }

    class GiatKrematorium {
        +index()
        +tambah()
        +simpan()
        +edit(id)
        +update()
        +hapus(id)
    }

    class Tagihan {
        +index()
        +generate(NIK)
        +bayar(id_tagihan)
        +lunas(id_tagihan)
        +tolak(id_tagihan)
        +cetak_surat(id_tagihan)
        +cetak_bukti_lunas(id_tagihan)
    }

    class Laporan {
        +pendapatan()
        +pendapatanHari()
        +pembayaran()
        +cetak_pdf()
    }

    class Pencarian {
        +index()
        +cari()
        +detail(NIK)
    }

    class MasterData {
        +makam()
        +krematorium()
        +pejabat()
        +pengguna()
        +petugas()
        +sarana()
        +berita()
    }

    class LogActivity {
        +index()
        +filter()
    }

    class Cetak {
        +almarhum_pdf()
        +almarhum_krematorium_pdf()
        +all_almarhum_pdf()
        +kompensasi_pdf()
    }

    class Profil {
        +index()
        +edit()
        +update_password()
    }

    %% --- MODELS ---
    class Model_Operasional {
        +getAll(table)
        +insert(table, data)
        +update(table, data, where)
        +delete(table, where)
        +getById(table, id)
        +query(sql)
    }

    class Model_Almarhum {
        +get_all()
        +get_by_nik(NIK)
        +insert_almarhum(data)
        +update_almarhum(data, NIK)
        +delete_almarhum(NIK)
        +cek_nik(NIK)
        +get_ahli_waris(NIK)
        +insert_ahli_waris(data)
        +update_isi_makam(ID_ISI, status)
        +get_riwayat_pengambilan()
    }

    class Model_AlmarhumKrematorium {
        +get_all()
        +get_by_nik(NIK)
        +insert(data)
        +update(data, NIK)
        +delete(NIK)
    }

    class Model_GiatMakam {
        +get_all()
        +get_by_tanggal(tgl)
        +get_range(tgl1, tgl2)
        +insert(data)
        +update(data, id)
        +delete(id)
        +upload_foto(id, file)
    }

    class Model_GiatKrematorium {
        +get_all()
        +get_by_tanggal(tgl)
        +insert(data)
        +update(data, id)
        +delete(id)
    }

    class Model_Tagihan {
        +generate_tagihan(NIK)
        +get_tagihan_by_nik(NIK)
        +get_all_tagihan()
        +update_status_lunas(id)
        +update_sisa_tagihan(id, sisa)
        +get_detail_tagihan(id)
    }

    class Model_Pembayaran {
        +insert(data)
        +get_by_tagihan(id_tagihan)
        +get_range(tgl1, tgl2)
    }

    class Model_GiatLaporan {
        +get_pendapatan_range(tgl1, tgl2)
        +get_pendapatan_hari(tgl)
        +get_total_pendapatan(tgl1, tgl2)
        +get_rekap_giat(tgl1, tgl2)
    }

    class Model_Makam {
        +get_all()
        +get_by_id(id)
        +insert(data)
        +update(data, id)
        +delete(id)
        +get_kapasitas_blok(id_makam)
        +get_isi_makam(id_makam)
    }

    class Model_Pengguna {
        +cek_login(username, password)
        +get_all()
        +insert(data)
        +update(data, id)
        +delete(id)
    }

    class Model_Pencarian {
        +cari_by_nama(nama)
        +cari_by_nik(NIK)
        +cari_by_tanggal(tgl)
        +cari_by_makam(id_makam)
        +cari_advanced(filters)
    }

    class Model_Chart {
        +get_statistik_makam()
        +get_statistik_kremasi()
        +get_tren_pendapatan(tahun)
    }

    class Model_Cetak_PDF {
        +get_almarhum_pdf(NIK)
        +get_all_almarhum_pdf()
        +get_kompensasi_pdf(id)
        +get_almarhum_krematorium_pdf(NIK)
    }

    class Model_Kompensasi {
        +get_all()
        +insert(data)
        +get_by_almarhum(NIK)
    }

    class Model_Pejabat {
        +get_all()
        +get_ttd()
        +insert(data)
        +update(data, id)
    }

    class Model_Sarana {
        +get_all()
        +get_by_makam(id_makam)
        +insert(data)
        +update(data, id)
    }

    class Model_Kecamatan {
        +get_all()
        +get_kelurahan(id_kec)
    }

    class Model_XTagihan {
        +get_tagihan_orang(id_tagihan)
        +insert(data)
        +update(data, id)
    }

    %% --- LIBRARIES ---
    class Auth {
        +is_logged_in()
        +login(user, pass)
        +logout()
        +set_session(data)
        +get_user()
        +cek_role(role)
    }

    class Template {
        +set(key, value)
        +load(template, view, data)
        +render()
    }

    class PDF_Generator {
        +generate_pdf(data, title)
        +set_kop_surat(data)
        +add_ttd(pejabat)
        +output(filename)
    }

    %% --- RELATIONSHIPS ---
    Home --|> CI_Controller
    AlmarhumExt --|> CI_Controller
    GiatMakam --|> CI_Controller
    GiatKrematorium --|> CI_Controller
    Tagihan --|> CI_Controller
    Laporan --|> CI_Controller
    Pencarian --|> CI_Controller
    MasterData --|> CI_Controller
    LogActivity --|> CI_Controller
    Cetak --|> CI_Controller
    Profil --|> CI_Controller

    Model_Almarhum --|> CI_Model
    Model_AlmarhumKrematorium --|> CI_Model
    Model_GiatMakam --|> CI_Model
    Model_GiatKrematorium --|> CI_Model
    Model_Tagihan --|> CI_Model
    Model_Pembayaran --|> CI_Model
    Model_GiatLaporan --|> CI_Model
    Model_Makam --|> CI_Model
    Model_Pengguna --|> CI_Model
    Model_Pencarian --|> CI_Model
    Model_Chart --|> CI_Model
    Model_Cetak_PDF --|> CI_Model
    Model_Kompensasi --|> CI_Model
    Model_Pejabat --|> CI_Model
    Model_Sarana --|> CI_Model
    Model_Kecamatan --|> CI_Model
    Model_XTagihan --|> CI_Model
    Model_Operasional --|> CI_Model

    Home --> Auth : uses
    Home --> Template : uses
    AlmarhumExt --> Model_Almarhum : uses
    AlmarhumExt --> Model_Makam : uses
    AlmarhumExt --> Model_Kompensasi : uses
    GiatMakam --> Model_GiatMakam : uses
    GiatKrematorium --> Model_GiatKrematorium : uses
    Tagihan --> Model_Tagihan : uses
    Tagihan --> Model_Pembayaran : uses
    Tagihan --> Model_XTagihan : uses
    Laporan --> Model_GiatLaporan : uses
    Pencarian --> Model_Pencarian : uses
    Cetak --> Model_Cetak_PDF : uses
    Cetak --> Model_Pejabat : uses
    MasterData --> Model_Makam : uses
    MasterData --> Model_Pengguna : uses
    MasterData --> Model_Pejabat : uses
    MasterData --> Model_Sarana : uses
    Profil --> Model_Pengguna : uses
```

### 3.3 Modul Dependency Map

```mermaid
graph TD
    subgraph "🌐 Presentation Layer"
        VIEWS["Views (Bootstrap + jTable)"]
        TEMPLATE["Template.php / Templatev2.php"]
    end

    subgraph "🎮 Controller Layer (15 Controllers)"
        HOME["Home\n(auth + dashboard)"]
        ALM["AlmarhumExt\n(CRUD almarhum)"]
        G_MKM["GiatMakam\n(kegiatan makam)"]
        G_KRM["GiatKrematorium\n(kegiatan kremasi)"]
        G_LAP["GiatLaporan\n(rekap + chart)"]
        G_TGT["GiatTarget\n(target tahunan)"]
        G_TM["GiatTarifMakam\n(tarif makam)"]
        G_TK["GiatTarifKrematorium\n(tarif kremasi)"]
        TAGIHAN["Tagihan\n(keuangan)"]
        LAPORAN["Laporan\n(pendapatan)"]
        CARI["Pencarian\n(search engine)"]
        CETAK["Cetak\n(PDF generator)"]
        MASTER["MasterData\n(kelola referensi)"]
        LOG_C["Log\n(audit trail)"]
        PROFIL["Profil\n(user settings)"]
    end

    subgraph "🗄️ Model Layer (30 Models)"
        M_ALM["Model_Almarhum"]
        M_ALMK["Model_AlmarhumKrematorium"]
        M_GM["Model_GiatMakam"]
        M_GK["Model_GiatKrematorium"]
        M_GL["Model_GiatLaporan"]
        M_TAG["Model_Tagihan"]
        M_BAYAR["Model_Pembayaran"]
        M_XTAG["Model_XTagihan"]
        M_MAKAM["Model_Makam"]
        M_BLOK["Model_BlokKapasitas"]
        M_KREM["Model_Krematorium"]
        M_COMP["Model_Kompensasi"]
        M_CARI["Model_Pencarian"]
        M_CPDF["Model_Cetak_* (4 models)"]
        M_CHART["Model_Chart"]
        M_PENGGUNA["Model_Pengguna"]
        M_PETUGAS["Model_Petugas"]
        M_PEJABAT["Model_Pejabat"]
        M_SARANA["Model_Sarana"]
        M_KEC["Model_Kecamatan"]
        M_BERITA["Model_Berita"]
        M_OP["Model_Operasional (base)"]
    end

    subgraph "🛠️ Libraries / Helpers"
        AUTH["Auth Library\n(session + role)"]
        PDF_LIB["PDF Library\n(TCPDF)"]
        FORM_VAL["Form Validation"]
    end

    subgraph "🗃️ Database"
        DB[("MariaDB\ndkp_swag\n(29 tables)")]
    end

    VIEWS --> TEMPLATE
    TEMPLATE --> HOME
    TEMPLATE --> ALM
    TEMPLATE --> G_MKM
    TEMPLATE --> TAGIHAN

    HOME --> AUTH
    HOME --> M_OP
    HOME --> M_BERITA

    ALM --> M_ALM
    ALM --> M_MAKAM
    ALM --> M_COMP
    ALM --> M_KEC

    G_MKM --> M_GM
    G_MKM --> M_OP

    G_KRM --> M_GK
    G_KRM --> M_KREM

    G_LAP --> M_GL
    G_LAP --> M_CHART

    G_TGT --> M_OP
    G_TM --> M_OP
    G_TK --> M_OP

    TAGIHAN --> M_TAG
    TAGIHAN --> M_BAYAR
    TAGIHAN --> M_XTAG
    TAGIHAN --> M_ALM

    LAPORAN --> M_GL
    LAPORAN --> M_BAYAR

    CARI --> M_CARI
    CARI --> M_ALM

    CETAK --> M_CPDF
    CETAK --> M_PEJABAT

    MASTER --> M_MAKAM
    MASTER --> M_BLOK
    MASTER --> M_KREM
    MASTER --> M_PENGGUNA
    MASTER --> M_PETUGAS
    MASTER --> M_PEJABAT
    MASTER --> M_SARANA
    MASTER --> M_BERITA

    LOG_C --> M_OP
    PROFIL --> M_PENGGUNA

    M_ALM --> DB
    M_GM --> DB
    M_TAG --> DB
    M_BAYAR --> DB
    M_MAKAM --> DB
    M_PENGGUNA --> DB
    M_OP --> DB
    M_CPDF --> DB

    AUTH --> M_PENGGUNA
    M_PENGGUNA --> DB
```

### 3.4 Alur Data — Flow Diagram

```mermaid
flowchart LR
    subgraph INPUT["📥 Input"]
        F1["Form Almarhum\n(NIK, Nama, Tgl, Plot)"]
        F2["Form Giat Harian\n(Tanggal, Blok, Jumlah)"]
        F3["Form Pembayaran\n(Tagihan, Jumlah, Metode)"]
        F4["Upload Dokumen\n(KTP, SKTM, Foto)"]
    end

    subgraph PROCESS["⚙️ Process"]
        P1["Validasi & Cek Duplikasi"]
        P2["Hitung Retribusi & Denda"]
        P3["Update Status Makam"]
        P4["Generate No. Tagihan"]
        P5["Catat Log Aktivitas"]
        P6["Generate PDF Laporan"]
    end

    subgraph STORAGE["🗄️ Storage"]
        DB1[("almarhum\ntagihan\npembayaran")]
        DB2[("giat_makam\ngiat_krematorium\nlog_activity")]
        FILE[("File System\nKTP/ SKTM/\nFoto/Bukti")]
    end

    subgraph OUTPUT["📤 Output"]
        O1["📊 Dashboard\nStatistik"]
        O2["📄 Surat Tagihan\nBukti Lunas"]
        O3["📈 Laporan\nPendapatan"]
        O4["🔍 Hasil\nPencarian"]
        O5["🖨️ PDF\nDokumen Resmi"]
    end

    F1 --> P1 --> P3 --> DB1
    F2 --> P5 --> DB2
    F3 --> P2 --> P4 --> DB1
    F4 --> FILE

    DB1 --> P6 --> O2
    DB1 --> P6 --> O3
    DB1 --> P6 --> O5
    DB2 --> O1
    DB1 --> O4
    DB2 --> O3
```

---

## 📊 Ringkasan Blueprint

| Diagram                       | Cakupan                                                                     |
| ----------------------------- | --------------------------------------------------------------------------- |
| **Use Case Diagram**    | 3 level: High-level (13 use case, 4 aktor), Detail Almarhum, Detail Tagihan |
| **Sequence Diagram**    | 4 alur: Registrasi Almarhum, Pembayaran, Cetak Laporan, Autentikasi         |
| **Domain Model (ERD)**  | 29 tabel lengkap dengan relasi, primary key, foreign key, dan atribut       |
| **Class Diagram (MVC)** | 15 Controller, 30 Model, 3 Library, lengkap dengan method signature         |
| **Dependency Map**      | Arsitektur 4-layer: Presentation → Controller → Model → Database         |
| **Flow Diagram**        | Alur data end-to-end: Input → Process → Storage → Output                 |

---

> **Blueprint ini dibuat oleh Giant Rachman Junaedi untuk menggambarkan arsitektur lengkap DKP SWAG** dari sisi bisnis (use case), alur interaksi (sequence), struktur data (domain model/ERD), organisasi kode (class diagram), hingga dependensi antar modul.
