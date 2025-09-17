## Product Requirements Document (PRD) - Aplikasi Manajemen Rumah Makan (MVP)

- **Versi**: 1.0
- **Tanggal**: 17 September 2025
- **Penyusun**: Muhamad Azkal Fahmi Yahya_221240001336 dan Khaf Sari Inayati_221240001228

### 1. Pendahuluan
#### 1.1 Visi Produk
Mewujudkan platform yang intuitif dan terjangkau untuk membantu pemilik dan manajer rumah makan kecil hingga menengah mengelola operasional sehari-hari mereka. Visi kami adalah menyederhanakan manajemen menu, inventaris, proses penjualan (POS), dan menyediakan laporan dasar, memberdayakan rumah makan untuk fokus pada penyajian makanan berkualitas dan pengalaman pelanggan yang luar biasa, tanpa terbebani kerumitan teknis.

#### 1.2 Tujuan Produk (MVP)
Tujuan utama dari Minimum Viable Product (MVP) ini adalah untuk memvalidasi kebutuhan pasar akan solusi manajemen rumah makan yang berbasis SQL lokal untuk setiap restoran, menawarkan fungsionalitas inti yang esensial. Kami bertujuan untuk menyediakan alat yang kuat namun sederhana yang dapat digunakan oleh satu pengguna utama per rumah makan untuk mengelola aspek-aspek kunci bisnis mereka, serta mengumpulkan umpan balik pengguna awal untuk iterasi di masa mendatang.

#### 1.3 Target Pengguna
- **Pemilik/Manajer Rumah Makan Kecil & Menengah**: Mencari solusi all-in-one yang mudah digunakan untuk mengelola menu, stok, dan penjualan; membutuhkan antarmuka intuitif.
- **Wirausahawan Kuliner**: Membutuhkan sistem manajemen efisien dan terjangkau untuk mendukung pertumbuhan bisnis.
- **Super Admin (Tim Pengembangan/Anda)**: Mengelola platform, pendaftaran restoran baru, monitoring langganan, pemeliharaan backend.

#### 1.4 Latar Belakang & Masalah yang Diselesaikan
Banyak rumah makan kecil masih mengandalkan pencatatan manual atau sistem yang terfragmentasi untuk mengelola operasional. Produk ini mengatasi masalah dengan platform terintegrasi yang:
- Memusatkan data menu, inventaris, dan penjualan.
- Mengotomatiskan pengurangan stok berdasarkan penjualan.
- Menyediakan laporan penjualan dasar untuk keputusan lebih baik.
- Memanfaatkan database SQL lokal untuk kontrol data dan kinerja, tetap terhubung ke langganan/update.

### 2. Fitur MVP
MVP fokus pada 6 alur kerja inti dengan dukungan database SQL lokal per restoran.

#### 2.1 Pendaftaran Restoran & Manajemen Akun Pengguna Utama 
- **Kemampuan**:
  - Pengguna mendaftarkan restoran baru dengan email dan password.
  - Setiap pendaftaran membuat instance restoran baru beserta alokasi database SQL lokal di sisi server.
  - Pengguna dapat login dengan akun utama mereka.
  - Pengguna dapat mengedit informasi dasar restoran (nama, alamat, kontak).
- **Catatan**: Data pendaftaran dan pengguna utama disimpan di database pusat; aplikasi akan mengidentifikasi database SQL lokal yang sesuai saat login.
- **Manfaat**: Memungkinkan multi-tenant dengan data terisolasi.

#### 2.2 Manajemen Menu & Kategori
- **Kemampuan**:
  - CRUD kategori menu (contoh: Makanan Utama, Minuman, Hidangan Pembuka).
  - CRUD item menu dengan nama, deskripsi, harga jual, kategori.
  - Unggah foto item menu (disimpan di storage server terhubung dengan DB lokal restoran).
- **Manfaat**: Memudahkan pengelolaan penawaran produk.

#### 2.3 Manajemen Resep
- **Kemampuan**: Membuat resep untuk tiap item menu, menghubungkan dengan bahan baku dan kuantitas.
- **Manfaat**: Pengurangan stok otomatis, mendukung perhitungan COGS (tanpa pencatatan akuntansi formal di MVP).

#### 2.4 Manajemen Inventaris Bahan Baku
- **Kemampuan**:
  - CRUD bahan baku (nama, SKU, satuan ukur, harga beli per satuan, stok awal).
  - Melihat daftar bahan baku beserta sisa stok.
  - Pengurangan stok otomatis berdasarkan resep yang terjual.
  - Penyesuaian stok manual (tambah/kurang).
- **Manfaat**: Menjaga ketersediaan bahan baku dan melacak nilai stok.

#### 2.5 Modul Point of Sales (POS) / Kasir
- **Kemampuan**:
  - Memulai transaksi baru.
  - Menambahkan/mengubah/menghapus item transaksi.
  - Hitung total otomatis, terima pembayaran tunai, hitung kembalian.
  - Simpan transaksi ke database SQL lokal.
  - Cetak struk sederhana dan lihat riwayat transaksi.
- **Manfaat**: Alat kasir efisien untuk operasional harian.

#### 2.6 Laporan Dasar Penjualan
- **Kemampuan**:
  - Laporan total penjualan per periode (harian, mingguan, bulanan).
  - Laporan item menu terlaris per periode.
- **Manfaat**: Wawasan performa penjualan.

### 3. Fitur yang Tidak Termasuk dalam MVP ini
- Manajemen Meja, tata letak meja, pemesanan per meja.
- Akuntansi dasar detail (jurnal otomatis, Laba Rugi formal, Neraca).
- Manajemen langganan otomatis (dilakukan manual/eksternal untuk MVP).
- Manajemen multi-user internal per restoran.
- Metode pembayaran non-tunai di POS.
- Diskon/Promosi kompleks atau voucher.
- Integrasi pihak ketiga (payment gateway, aplikasi pesan antar).
- Fungsionalitas offline POS canggih.
- Backup & Restore data oleh pengguna.
- Kustomisasi template struk mendalam.

### 4. Arsitektur Data (Penyesuaian untuk SQL Lokal)
- **Database Pusat (Backend)**
  - Tujuan: Mengelola informasi pendaftaran restoran dan status langganan.
  - Contoh Teknologi: PostgreSQL, MySQL, atau SQL Server.
  - Tabel Kunci: `users`, `restaurants`.
- **Database SQL Lokal (Per Restoran)**
  - Tujuan: Menyimpan data operasional (menu, inventaris, penjualan, resep) terisolasi per restoran.
  - Implementasi: Alokasikan skema/database SQL baru per restoran, simpan detail koneksi di database pusat.
  - Tabel Kunci: `menu_categories`, `menu_items`, `ingredients`, `recipe_details`, `sales_transactions`, `sales_transaction_details`.

### 5. Kebutuhan Non-Fungsional Utama
- **Isolasi Data**: Data tiap restoran terisolasi via skema/database SQL terpisah.
- **Keamanan**: Otentikasi aman; akses DB lokal via backend; enkripsi data sensitif.
- **Performa**: Responsif; query dioptimalkan.
- **Skalabilitas**: Desain multi-tenant dapat tumbuh dengan jumlah restoran dan data.
- **Ketersediaan & Keandalan**: Ketersediaan tinggi; strategi backup/restore.
- **Usability**: UI Flutter intuitif untuk pengguna non-teknis.

### 6. Metrik Keberhasilan (MVP)
- **Adopsi**: Minimal X restoran aktif dalam Y bulan pertama.
- **Retensi**: Z% pengguna aktif kembali setelah 1 bulan.
- **Kualitas Data**: Selisih stok < 5%.
- **Kepuasan Pengguna**: Skor kepuasan W.
- **Fungsionalitas**: Semua fitur MVP berjalan tanpa critical bugs.

### 7. Rencana Rilis & Timeline (Ringkasan)
- **Fase 1 (MVP)**: Pengembangan dan peluncuran fitur inti.
- **Target Rilis**: [Tanggal Target - Misal: Q4 2025]
- **Metodologi**: Agile (Sprint-based)
- **Fokus**: Validasi pasar, kumpulkan umpan balik, stabilitas sistem inti.

