# Perampingan Ekstrem & Optimasi Performa Kafelas Laundry

Transformasi performa menyeluruh untuk memangkas ukuran aplikasi, menghapus berkas arsip biner berlebih, menghapus halaman statis luar yang tidak terpakai, dan menggantikan pustaka grafik berat dengan tabel data ringkas berkecepatan tinggi.

---

### User Review & Critical Decisions

> [!IMPORTANT]
> Berdasarkan konfirmasi pengguna pada sesi tanya jawab:
> 1. **Berkas Arsip & Aset**: Seluruh berkas ZIP besar (`dist.zip`, `kflsagnd-kasir-pc.zip`), file master Affinity (`logo kafelas laundry.af`), serta gambar demo yang usang akan dihapus dari repositori.
> 2. **Halaman Statis Luar**: Seluruh halaman mandiri di luar POS kasir (`landing.html`, `booking.html`, `profil.html`, `tutorial.html`) dihapus sepenuhnya agar aplikasi 100% berfokus sebagai Kasir POS Laundry yang ringan dan cepat.
> 3. **Grafik & Animasi Berat**: Visual grafik D3 (`d3`) dan engine animasi (`motion`, `canvas-confetti`) dihapus dari bundle. Seluruh visualisasi diganti dengan **tabel data angka ringkas** (`tabular-nums`) yang bersih, responsif, dan langsung terbuka seketika (0ms delay).

- **Confirmed Decision 1**: Hapus file ZIP besar dan aset biner tidak terpakai di folder `public/`.
- **Confirmed Decision 2**: Hapus seluruh halaman HTML statis eksternal (`landing.html`, `booking.html`, `profil.html`, `tutorial.html`) beserta denylist-nya di konfigurasi Service Worker.
- **Confirmed Decision 3**: Hapus dependency `d3`, `@types/d3`, `motion`, `canvas-confetti`, dan ganti chart di Dashboard & Laporan dengan ringkasan matriks tabel tabular angka berpresisi tinggi.

---

### 1. Overview & Core Concept

- **Tujuan Utama**: Menghasilkan aplikasi kasir laundry dengan waktu muat (First Contentful Paint) secepat kilat, konsumsi memori minim, dan hemat penyimpanan perangkat kasir/HP.
- **Target Pengguna**: Pemilik (Owner) dan Kasir Laundry yang membutuhkan sistem POS instan, tanpa beban loading grafik lambat atau unduhan cache yang membengkak.
- **Nilai Tambah**: Pengurangan ukuran bundle JS hingga lebih dari 60%, hilangnya puluhan megabyte berkas sampah dari `public/`, dan stabilitas offline-first yang jauh lebih tangguh.

---

### 2. User Experience & Visual Design

- **Alur Pengguna Pasca-Optimasi**:
  1. Pengguna membuka URL aplikasi atau shortcut PWA di HP/PC.
  2. Aplikasi memuat instan (sub-second load) langsung ke alur Kasir Laundry (Splash/Auth/Dashboard).
  3. Di **Dashboard**: Ringkasan omset dan tren disajikan dalam bentuk kartu ringkas berangka tabular monokrom/emerald, tanpa menunggu kalkulasi visual SVG D3.
  4. Di **Laporan Keuangan**: Komparasi bulanan/tahunan disajikan melalui tabel matriks perbandingan angka persentase delta (+/-%) yang padat, rapi, dan mudah diaudit.
  5. Seluruh dialog dan transisi antar halaman berpindah seketika tanpa stutter atau frame drop dari animasi JavaScript berat.

- **Visual Identity & Theme**:
  - *Palet Warna*: Dominan kanvas putih/slate netral (60%), kartu flat bergaris batas halus `border-slate-200` (30%), dan aksen marun signature `#800000` (10%).
  - *Tipografi*: Angka keuangan, nota, dan metrik menggunakan format `font-mono tabular-nums` untuk perataan vertikal sempurna.
  - *Tabel & Komparasi*: Desain baris tabel kompak ($36\text{px}$–$44\text{px}$) dengan teks rata kiri dan nominal uang rata kanan.

---

### 3. Key Product Decisions & Trade-Offs

- **Penghapusan D3 & Motion vs. Tabel Ringkas Native**:
  - *Pendekatan*: Menghapus library `d3` (ukuran ~250KB minified) dan `motion` (~100KB minified).
  - *Alasan*: Aplikasi kasir operasional mengutamakan kecepatan dan kepraktisan membaca angka real transaksi (omset, piutang, kas) daripada grafik lengkung yang berat di perangkat HP berspesifikasi rendah.
  - *Alternatif yang Ditinggalkan*: Mempertahankan grafik Canvas/SVG custom — ditolak karena tabel angka jauh lebih cepat, hemat baterai, dan mudah dibaca kasir.

- **Pembersihan Halaman Statis Eksternal**:
  - *Pendekatan*: Menghapus `landing.html`, `booking.html`, `profil.html`, `tutorial.html`.
  - *Alasan*: Fitur web profil studio foto atau landing page lama sudah tidak relevan dengan fokus inti sistem POS Kafelas Laundry. Menghapus halaman ini menyederhanakan konfigurasi PWA Workbox dan menghemat kuota cache.

- **Pembersihan File ZIP & Master Desain**:
  - *Pendekatan*: Menghapus `public/dist.zip`, `public/kflsagnd-kasir-pc.zip`, dan `public/logo kafelas laundry.af`.
  - *Alasan*: Berkas-berkas biner ini menghabiskan puluhan megabyte ruang disk dan memperlambat sinkronisasi serta build deployment.

---

### 4. Technical Architecture & Data Strategy

```
┌─────────────────────────────────────────────────────────────┐
│                 Kafelas Laundry - Ultra-Light POS           │
└──────────────────────────────┬──────────────────────────────┘
                               │
               ┌───────────────┴───────────────┐
               ▼                               ▼
    ┌──────────────────────┐        ┌──────────────────────┐
    │    Ultra-Fast POS    │        │  Tabular Financial   │
    │      Dashboard       │        │       Reports        │
    │  (CSS Tabular Cards) │        │   (Clean Data Grid)  │
    └──────────┬───────────┘        └──────────┬───────────┘
               │                               │
               └───────────────┬───────────────┘
                               │
                               ▼
    ┌──────────────────────────────────────────────────────────┐
    │   Zero-Dependency State & Storage Engine                 │
    │   - IndexedDB & LocalStorage Single Source of Truth      │
    │   - Real-time Firestore Sync (Background / Delta)        │
    │   - No Heavy Chart Engines (Zero D3, Zero Motion)        │
    └──────────────────────────────────────────────────────────┘
```

#### Komponen yang Disesuaikan:
1. **`src/components/BranchComparisonD3Chart.tsx`**:
   - Dirombak menjadi komponen ringkas berbasis tabel komparasi cabang (`BranchComparisonTable.tsx`) tanpa ketergantungan D3.
2. **`src/components/dashboard/DashboardMiniTrendChart.tsx`**:
   - Disederhanakan menjadi ringkasan angka tren persentase delta (naik/turun) dengan indikator panah ringkas SVG inline.
3. **`src/views/LaporanView.tsx`**:
   - Menghapus grafik D3/Canvas kompleks, mempertahankan tabel rincian transaksi, arus kas, dan komparasi persentase omset bulanan yang responsif.
4. **`package.json`**:
   - Menghapus `d3`, `@types/d3`, `motion`, `canvas-confetti`, `@types/canvas-confetti`.
5. **`public/` & `vite.config.ts`**:
   - Menghapus berkas ZIP besar, berkas `.af`, dan berkas `.html` luar.
   - Merapikan `navigateFallbackDenylist` pada konfigurasi Vite PWA.

---

### Verification Plan

1. **Pembersihan File Sampah**: Memastikan seluruh berkas ZIP, master `.af`, dan berkas HTML luar berhasil dihapus dari direktori proyek.
2. **Uji Dependensi**: Menjalankan build TypeScript untuk memastikan tidak ada import usang (`d3`, `motion`, dsb.).
3. **Uji Kompilasi (`compile_applet`)**: Memverifikasi aplikasi terkompilasi 100% bebas error dengan ukuran bundle yang jauh lebih kecil.
4. **Uji Fungsionalitas Kasir**: Memastikan alur transaksi baru, pembukuan kas, status pengerjaan, cetak nota struk, dan ekspor data tetap bekerja normal tanpa kendala.
