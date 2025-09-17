## Checklist Timeline Sprint MVP - Aplikasi Manajemen Rumah Makan (SQL Lokal)

- **Versi**: 1.0
- **Tanggal**: 28 Mei 2025
- **Penyusun**: Muhamad Azkal Fahmi Yahya_221240001336 dan Khaf Sari Inayati_221240001228

### Metodologi
- Pendekatan: Agile Development (Sprint-based)
- Durasi Sprint: 2 Minggu per Sprint
- Fokus MVP: Fungsionalitas inti yang dijelaskan dalam PRD dan SRS, dengan arsitektur SQL Lokal Multi-Tenant.
- Prioritas: Fitur kritikal untuk validasi pasar dan operasional minimum.

### Asumsi
- Tim pengembangan memiliki setidaknya 1-2 Developer Frontend (Flutter) dan 1-2 Developer Backend (untuk API dan manajemen DB SQL).
- Lingkungan development dan testing (DB, server backend) sudah tersedia atau dapat disiapkan dengan cepat.
- Tidak ada hambatan besar yang tidak terduga dalam integrasi teknologi.

### Sprint 1: Setup Dasar & Autentikasi (2 Minggu)
- Fokus: Menyiapkan fondasi arsitektur multi-tenant SQL, registrasi, dan login pengguna.

#### Backend & Database (SQL Lokal)
- [ ] Inisialisasi proyek Backend (pilihan teknologi: Node.js/Python/Go/Java).
- [ ] Setup Central PostgreSQL/MySQL DB.
- [ ] Buat tabel USERS (id, email, password_hash, name, is_super_admin, created_at, updated_at).
- [ ] Buat tabel RESTAURANTS (id, restaurant_name, primary_user_id (FK), db_connection_string, subscription_status, created_at, updated_at).
- [ ] Implementasi Endpoint Backend API:
  - [ ] POST /auth/register (Registrasi Pengguna Utama & Restoran Baru):
    - [ ] Buat USERS entry.
    - [ ] Logika pembuatan/alokasi skema/database SQL lokal baru.
    - [ ] Logika inisialisasi skema (membuat tabel kosong untuk Restoran-Specific DBs).
    - [ ] Buat RESTAURANTS entry dengan db_connection_string.
    - [ ] Generate & kembalikan JWT.
  - [ ] POST /auth/login (Login Pengguna Utama):
    - [ ] Verifikasi kredensial.
    - [ ] Generate & kembalikan JWT dengan user_id dan restaurant_id (diambil dari RESTAURANTS tabel).
- [ ] Middleware Autentikasi (Validasi JWT & Ekstraksi restaurant_id).
- [ ] Middleware Multi-Tenant (Menggunakan restaurant_id dari JWT untuk memilih db_connection_string yang tepat untuk operasi DB).

#### Frontend (Flutter App)
- [ ] Inisialisasi proyek Flutter.
- [ ] Setup arsitektur aplikasi (struktur folder, state management dasar).
- [ ] Implementasi halaman "Splash Screen".
- [ ] Implementasi halaman "Registrasi Restoran".
- [ ] Form input (Nama Restoran, Email, Password).
- [ ] Panggilan API POST /auth/register.
- [ ] Implementasi halaman "Login Pengguna Utama".
- [ ] Form input (Email, Password).
- [ ] Panggilan API POST /auth/login.
- [ ] Penyimpanan token JWT di lokal (shared_preferences/Hive).
- [ ] Navigasi dasar setelah login/registrasi berhasil ke halaman Dashboard (placeholder).

### Sprint 2: Manajemen Menu & Inventaris Dasar (2 Minggu)
- [ ] Fokus: Memungkinkan restoran untuk mengelola menu dan bahan baku mereka.

#### Backend & Database (SQL Lokal)
- [ ] Update skema Restaurant-Specific DB (jika diperlukan untuk penyesuaian).
- [ ] Tambah tabel MENU_CATEGORIES.
- [ ] Tambah tabel MENU_ITEMS.
- [ ] Tambah tabel INGREDIENTS.
- [ ] Implementasi Endpoint Backend API untuk Manajemen Menu:
  - [ ] CRUD untuk MENU_CATEGORIES.
  - [ ] CRUD untuk MENU_ITEMS (tanpa upload gambar dulu).
- [ ] Implementasi Endpoint Backend API untuk Manajemen Inventaris:
  - [ ] CRUD untuk INGREDIENTS.
  - [ ] PUT /inventory/ingredients/:id/stock (Penyesuaian stok manual).

#### Frontend (Flutter App)
- [ ] Implementasi halaman "Dashboard" (tampilan awal setelah login).
- [ ] Implementasi halaman "Manajemen Kategori Menu".
- [ ] UI daftar kategori.
- [ ] Form tambah/edit/hapus kategori.
- [ ] Integrasi API Kategori.
- [ ] Implementasi halaman "Manajemen Item Menu".
- [ ] UI daftar item menu.
- [ ] Form tambah/edit/hapus item menu (tanpa upload gambar).
- [ ] Integrasi API Item Menu.
- [ ] Implementasi halaman "Manajemen Inventaris Bahan Baku".
- [ ] UI daftar bahan baku.
- [ ] Form tambah/edit/hapus bahan baku.
- [ ] Form penyesuaian stok manual.
- [ ] Integrasi API Inventaris.

### Sprint 3: Resep & Integrasi Stok Otomatis (2 Minggu)
- [ ] Fokus: Menghubungkan menu ke bahan baku dan mengotomatiskan pengurangan stok.

#### Backend & Database (SQL Lokal)
- [ ] Update skema Restaurant-Specific DB.
- [ ] Tambah tabel RECIPE_DETAILS.
- [ ] Implementasi Endpoint Backend API untuk Manajemen Resep:
  - [ ] GET /menu-items/:menu_item_id/recipes.
  - [ ] POST /menu-items/:menu_item_id/recipes (untuk menambah/mengupdate resep).
- [ ] Penting: Modifikasi Endpoint POST /pos/transactions (akan dibuat di Sprint 4) untuk mencakup logika pengurangan stok otomatis berdasarkan RECIPE_DETAILS.

#### Frontend (Flutter App)
- [ ] Integrasi dengan penyimpanan gambar (cloud storage/lokal server) untuk item menu.
- [ ] Update form item menu untuk memungkinkan upload gambar.
- [ ] Menampilkan gambar di daftar item menu.
- [ ] Implementasi halaman "Manajemen Resep" (terkait dengan setiap item menu).
- [ ] UI untuk menampilkan resep suatu item menu.
- [ ] Form untuk memilih bahan baku dan menentukan kuantitas yang dibutuhkan.
- [ ] Integrasi API Resep.
- [ ] Review dan perbaikan UI/UX untuk modul Menu & Inventaris.

### Sprint 4: Modul POS & Laporan Dasar (2 Minggu)
- [ ] Fokus: Fungsionalitas inti kasir dan laporan penjualan pertama.

#### Backend & Database (SQL Lokal)
- [ ] Update skema Restaurant-Specific DB.
- [ ] Tambah tabel SALES_TRANSACTIONS.
- [ ] Tambah tabel SALES_TRANSACTION_DETAILS.
- [ ] Implementasi Endpoint Backend API untuk POS:
  - [ ] POST /pos/transactions (Menerima detail transaksi, menyimpan ke DB, mengurangi stok bahan baku otomatis).
  - [ ] GET /pos/transactions (Riwayat transaksi).
  - [ ] GET /pos/transactions/:id (Detail transaksi).
- [ ] Implementasi Endpoint Backend API untuk Laporan Dasar:
  - [ ] GET /reports/sales-summary (Total penjualan per periode).
  - [ ] GET /reports/top-selling (Item terlaris per periode).

#### Frontend (Flutter App)
- [ ] Implementasi halaman "Point of Sales (POS)".
- [ ] UI pemilihan item menu (dengan kategori).
- [ ] UI keranjang transaksi (daftar item, kuantitas, subtotal).
- [ ] Fungsionalitas mengubah kuantitas/menghapus item.
- [ ] Perhitungan total transaksi.
- [ ] Input pembayaran tunai & perhitungan kembalian.
- [ ] Panggilan API POST /pos/transactions.
- [ ] Implementasi fungsionalitas pencetakan struk sederhana (menggunakan plugin printer Flutter).
- [ ] Implementasi halaman "Riwayat Transaksi".
- [ ] UI daftar transaksi.
- [ ] Melihat detail transaksi.
- [ ] Integrasi API riwayat transaksi.
- [ ] Implementasi halaman "Laporan Penjualan".
- [ ] UI untuk melihat ringkasan penjualan (periode).
- [ ] UI untuk melihat item terlaris.
- [ ] Integrasi API Laporan.

### Sprint 5: Testing, Bug Fixing, & Deployment Persiapan (2 Minggu)
- [ ] Fokus: Memastikan stabilitas, kualitas, dan kesiapan untuk deployment.

#### Backend & Database (SQL Lokal)
- [ ] Unit Tests & Integration Tests untuk semua modul Backend API.
- [ ] Perbaikan bug yang ditemukan selama pengujian.
- [ ] Optimasi query database dan pembuatan indeks jika diperlukan.
- [ ] Konfigurasi lingkungan produksi (scaling, logging, monitoring).
- [ ] Implementasi strategi backup/restore database.

#### Frontend (Flutter App)
- [ ] Manual Testing end-to-end semua alur MVP.
- [ ] Perbaikan bug UI/UX dan fungsionalitas.
- [ ] Optimasi performa UI Flutter.
- [ ] Penanganan error dan pesan feedback yang user-friendly.
- [ ] Finalisasi aset aplikasi (ikon, splash screen).
- [ ] Persiapan untuk deployment ke Google Play Store (signing, konfigurasi).

### Pasca-MVP: Deployment & Iterasi Lanjutan
- [ ] Deployment MVP ke lingkungan produksi.
- [ ] Pengumpulan umpan balik dari pengguna awal.
- [ ] Perencanaan fitur-fitur tambahan untuk sprint berikutnya berdasarkan PRD dan "Rencana Pengembangan Berikutnya" di SRS.
