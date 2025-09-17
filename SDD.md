## System Design Document (SDD) - Aplikasi Manajemen Rumah Makan (MVP)

- **Versi**: 1.0
- **Tanggal**: 17 September 2025
- **Penyusun**: Muhamad Azkal Fahmi Yahya_221240001336 dan Khaf Sari Inayati_221240001228

### 1. Pendahuluan
#### 1.1 Tujuan
Dokumen ini menguraikan desain arsitektur dan komponen-komponen kunci dari Aplikasi Manajemen Rumah Makan untuk fase Minimum Viable Product (MVP). Ini akan mencakup gambaran sistem, struktur arsitektur, desain komponen utama, dan detail implementasi teknologi, dengan fokus pada penggunaan database SQL lokal untuk setiap tenant (restoran).
#### 1.2 Cakupan
SDD ini mencakup desain untuk:
- Arsitektur sistem secara keseluruhan (Frontend, Backend API, Database).
- Desain modul-modul utama (Autentikasi & Otorisasi, Manajemen Multi-Tenant, Menu, Inventaris, POS, Laporan).
- Pilihan teknologi dan justifikasinya.
- Desain API.
- Pertimbangan keamanan dan skalabilitas.

### 2. Gambaran Sistem
Sistem akan mengadopsi arsitektur microservices atau modular monolith, terdiri dari aplikasi frontend Flutter, backend API kustom yang stateless, dan dua lapisan database SQL: satu database pusat untuk manajemen, dan kumpulan database SQL lokal (atau skema terpisah) untuk data operasional setiap restoran.

```mermaid
graph TD
    A[Pengguna (Mobile App)] --> B[Flutter Frontend Application]
    B --> C[Backend API Service (Custom: Node.js/Python/Go/Java)]
    C --> D[Central PostgreSQL/MySQL DB]
    C --> E[Multiple Restaurant-Specific PostgreSQL/MySQL DBs (Isolated/Schema-Based)]

    subgraph Infrastruktur Cloud/Server
        C
        D
        E
    end
```

#### Alur Interaksi:
- Pengguna berinteraksi dengan Flutter Frontend Application di perangkat seluler mereka.
- Flutter Frontend berkomunikasi dengan Backend API Service melalui HTTP/HTTPS.
- Backend API Service menangani:
  - Autentikasi & Otorisasi menggunakan Central DB.
  - Manajemen Multi-Tenant: Mengidentifikasi restaurant_id dan db_connection_string dari pengguna yang login menggunakan Central DB.
  - Operasi Data Restoran: Mengarahkan permintaan CRUD ke Restaurant-Specific DBs yang sesuai.
  - Logika Bisnis: Mengimplementasikan semua aturan bisnis (misalnya, pengurangan stok otomatis, perhitungan total transaksi).
- Central DB menyimpan data pengguna utama, informasi pendaftaran restoran, dan detail langganan.
- Restaurant-Specific DBs menyimpan semua data operasional (menu, inventaris, penjualan) untuk setiap restoran secara terisolasi.

### 3. Desain Arsitektur
#### 3.1 Komponen Arsitektur Utama
Flutter Frontend Application:
- Platform: Android, iOS.
- Tujuan: Menyediakan antarmuka pengguna yang intuitif untuk interaksi dengan sistem.
- Teknologi: Flutter (Dart).
- Komponen Kunci:
  - UI/UX: Berbasis Material Design/Cupertino, menggunakan widget Flutter.
  - State Management: Provider, Riverpod, BLoC, atau GetX untuk mengelola status aplikasi.
  - Network Layer: Dio, http package untuk berkomunikasi dengan Backend API.
  - Local Storage: shared_preferences atau Hive untuk menyimpan token otentikasi dan konfigurasi lokal.
  - Printer Integration: Plugin Flutter untuk konektivitas printer thermal (Bluetooth, USB, Network).
Backend API Service:
- Tujuan: Menjadi jembatan antara Frontend dan Database, mengimplementasikan logika bisnis, dan mengelola keamanan.
- Teknologi: (Pilih salah satu)
  - Node.js + Express.js / NestJS
  - Python + Django / Flask
  - Go + Gin / Echo
  - Java + Spring Boot
- Database ORM/Driver: Sesuai bahasa dan framework yang dipilih (misal: Sequelize/TypeORM untuk Node.js, SQLAlchemy/Django ORM untuk Python, GORM untuk Go, Hibernate untuk Java).
- Komponen Kunci:
  - Authentication Module: Menangani registrasi, login, manajemen sesi (JWT).
  - Authorization Middleware: Memastikan pengguna yang terautentikasi hanya dapat mengakses data restoran mereka sendiri.
  - Multi-Tenant Router: Menganalisis token otentikasi/informasi sesi untuk mengidentifikasi restaurant_id dan memilih db_connection_string yang benar dari Central DB untuk setiap permintaan.
  - Business Logic Modules: Implementasi CRUD untuk Menu, Inventaris, Resep, POS, Laporan.
  - Database Interaction Layer: Berinteraksi dengan Central DB dan Restaurant-Specific DBs.
  - File Storage Service: Untuk menyimpan gambar menu (misal: S3-compatible storage, atau penyimpanan lokal server).
Central PostgreSQL/MySQL Database:
- Tujuan: Menyimpan data fundamental dan manajemen tenant.
- Teknologi: PostgreSQL atau MySQL (pilihan yang populer, handal, dan skalabel).
- Tabel Kunci: users, restaurants.
Multiple Restaurant-Specific PostgreSQL/MySQL Databases:
- Tujuan: Menyimpan data operasional yang terisolasi untuk setiap restoran.
- Implementasi Multi-Tenant:
  - Database per Tenant: Setiap restoran mendapatkan database fisik terpisah di server database Anda. Ini memberikan isolasi data terkuat.
  - Skema per Tenant: Satu database fisik besar, tetapi setiap restoran memiliki skema SQL-nya sendiri dengan semua tabel operasionalnya. Ini lebih efisien untuk manajemen sumber daya di awal.
  - Kolom Tenant ID: Satu database besar dengan semua tabel, dan setiap tabel memiliki kolom restaurant_id untuk memfilter data. (Kurang disarankan untuk keamanan dan skalabilitas di awal)
  - Pilihan Rekomendasi untuk MVP: Skema per Tenant (jika database mendukung) atau Database per Tenant untuk isolasi optimal.
- Tabel Kunci (di setiap database/skema): menu_categories, menu_items, ingredients, recipe_details, sales_transactions, sales_transaction_details.

#### 3.2 Modul Desain Utama
#### 3.2.1 Modul Otentikasi & Otorisasi
Autentikasi:
- Registrasi: Pengguna mendaftar dengan email dan password. Password di-hash (misal: bcrypt) sebelum disimpan di Central DB.users.
- Login: Pengguna memberikan email/password. Backend memverifikasi hash. Jika berhasil, JWT (JSON Web Token) dibuat dengan user_id dan restaurant_id sebagai payload. Token ini dikembalikan ke Flutter App.
- Sesi: Flutter App menyimpan JWT dan mengirimkannya di header Authorization untuk setiap permintaan API berikutnya.
Otorisasi:
- Middleware di Backend API akan memvalidasi JWT.
- Dari JWT, user_id dan restaurant_id akan diekstrak.
- Semua permintaan ke data operasional restoran akan dipastikan hanya mengakses Restaurant-Specific DB yang sesuai dengan restaurant_id di JWT.

#### 3.2.2 Modul Manajemen Multi-Tenant
Registrasi Restoran Baru:
- Backend menerima permintaan registrasi dari Flutter App (nama restoran, email, password).
- Membuat entri USERS baru di Central DB.
- Backend memicu proses otomatis untuk:
  - Membuat database SQL baru atau skema SQL baru (misal: schema_restoran_uuid) di server database Anda.
  - Menjalankan skrip inisialisasi skema (membuat tabel: menu_categories, menu_items, dst.) di database/skema baru ini.
  - Membuat entri RESTAURANTS baru di Central DB, menautkannya ke USERS dan menyimpan db_connection_string (atau identitas skema).
Resolusi Tenant: Untuk setiap permintaan API yang memerlukan akses ke data operasional restoran:
- Backend menerima JWT.
- Mengekstrak restaurant_id dari JWT.
- Mencari db_connection_string (atau identitas skema) yang terkait dengan restaurant_id ini di Central DB.
- Membuka koneksi database ke Restaurant-Specific DB yang sesuai dan menjalankan operasi yang diminta.

#### 3.2.3 Modul Manajemen Menu & Kategori
API Endpoints:
- GET /categories: Mengambil semua kategori menu.
- POST /categories: Menambahkan kategori baru.
- PUT /categories/:id: Mengupdate kategori.
- DELETE /categories/:id: Menghapus kategori.
- GET /menu-items: Mengambil semua item menu.
- POST /menu-items: Menambahkan item menu (termasuk upload gambar ke storage).
- PUT /menu-items/:id: Mengupdate item menu.
- DELETE /menu-items/:id: Menghapus item menu.
Logika Bisnis: Validasi input, relasi FK ke kategori.

#### 3.2.4 Modul Manajemen Resep
API Endpoints:
- GET /menu-items/:menu_item_id/recipes: Mengambil resep untuk item menu.
- POST /menu-items/:menu_item_id/recipes: Menambahkan/mengupdate resep (bisa berupa array ingredient_id dan quantity_needed).
Logika Bisnis: Memastikan menu_item_id dan ingredient_id valid dan milik restoran yang sama.

#### 3.2.5 Modul Manajemen Inventaris
API Endpoints:
- GET /ingredients: Mengambil semua bahan baku.
- POST /ingredients: Menambahkan bahan baku.
- PUT /ingredients/:id: Mengupdate bahan baku.
- DELETE /ingredients/:id: Menghapus bahan baku.
- POST /ingredients/:id/adjust-stock: Penyesuaian stok manual (penambahan/pengurangan).
Logika Bisnis: Validasi stok tidak negatif.

#### 3.2.6 Modul Point of Sales (POS)
API Endpoints:
- POST /sales-transactions: Menyelesaikan transaksi penjualan.
- GET /sales-transactions: Mengambil riwayat transaksi.
- GET /sales-transactions/:id: Mengambil detail transaksi.
Logika Bisnis (di endpoint POST /sales-transactions):
- Menerima daftar item yang terjual, kuantitas, harga, total pembayaran, uang tunai diterima.
- Memvalidasi item menu dan stok bahan baku yang cukup berdasarkan resep. Jika stok tidak cukup, transaksi bisa dibatalkan atau ditandai sebagai masalah.
- Merekam entri di SALES_TRANSACTIONS.
- Merekam entri di SALES_TRANSACTION_DETAILS untuk setiap item.
- Untuk setiap item yang terjual, ambil resepnya dari RECIPE_DETAILS.
- Kurangi current_stock dari INGREDIENTS berdasarkan quantity_sold * quantity_needed.
- Hitung cost_price_total_for_items berdasarkan harga beli bahan baku.

#### 3.2.7 Modul Laporan Dasar Penjualan
API Endpoints:
- GET /reports/sales-summary?start_date=...&end_date=...: Laporan total penjualan.
- GET /reports/top-selling-items?start_date=...&end_date=...: Laporan item terlaris.
Logika Bisnis: Query database SALES_TRANSACTIONS dan SALES_TRANSACTION_DETAILS dengan filter tanggal. Agregasi data untuk mendapatkan total dan peringkat.

### 4. Desain Antarmuka API (Contoh)
Base URL: https://api.yourdomain.com/v1
Header Otentikasi: Authorization: Bearer <JWT_TOKEN>
Metode	Endpoint	Deskripsi	Request Body (JSON)	Response (JSON)
POST	/auth/register	Registrasi restoran baru	{ "name": "...", "email": "...", "password": "..." }	{ "message": "Success", "token": "...", "restaurant_id": "..." }
POST	/auth/login	Login pengguna utama	{ "email": "...", "password": "..." }	{ "message": "Success", "token": "...", "restaurant_id": "..." }
GET	/restaurants/me	Ambil info restoran	-	{ "id": "...", "name": "...", ... }
PUT	/restaurants/me	Update info restoran	{ "name": "...", "address": "...", ... }	{ "message": "Success", "restaurant": { ... } }
GET	/menu/categories	Ambil daftar kategori	-	[{ "id": "...", "name": "...", ... }]
POST	/menu/categories	Tambah kategori	{ "name": "..." }	{ "message": "Success", "category": { ... } }
GET	/menu/items	Ambil daftar item menu	-	[{ "id": "...", "name": "...", "price": "...", ... }]
POST	/menu/items	Tambah item menu (multi-part form untuk gambar)	{ "name": "...", "price": ..., "categoryId": "...", "image": <file> }	{ "message": "Success", "item": { ... } }
POST	/menu/items/:id/recipes	Simpan resep untuk item menu	[{ "ingredientId": "...", "quantityNeeded": ... }]	{ "message": "Success" }
GET	/inventory/ingredients	Ambil daftar bahan baku	-	[{ "id": "...", "name": "...", "stock": ..., ... }]
POST	/inventory/ingredients	Tambah bahan baku	{ "name": "...", "unit": "...", "stock": ..., ... }	{ "message": "Success", "ingredient": { ... } }
PUT	/inventory/ingredients/:id/stock	Sesuaikan stok bahan baku (manual)	`{ "adjustment": ..., "type": "add"	"subtract" }`
POST	/pos/transactions	Selesaikan transaksi penjualan	{ "items": [{ "menuItemId": "...", "qty": ..., "price": ... }], "cashReceived": ..., "totalAmount": ... }	{ "message": "Success", "transactionId": "..." }
GET	/pos/transactions	Riwayat transaksi (filter: ?startDate=...&endDate=...)	-	[{ "id": "...", "totalAmountPaid": ..., ... }]
GET	/reports/sales-summary	Laporan penjualan (filter: ?startDate=...&endDate=...)	-	{ "totalTransactions": ..., "totalRevenue": ... }
GET	/reports/top-selling	Laporan item terlaris (filter: ?startDate=...&endDate=...)	-	[{ "menuItemId": "...", "name": "...", "qtySold": ... }]

### 5. Pertimbangan Non-Fungsional
5.1 Keamanan
Password Hashing: Gunakan algoritma hashing yang kuat (misal: bcrypt) untuk password.
JWT Security: Gunakan secret key yang kuat, set kadaluarsa token, dan implementasi refresh token (opsional untuk MVP).
HTTPS: Semua komunikasi antara Frontend dan Backend harus melalui HTTPS.
Input Validation: Validasi semua input dari Frontend di Backend untuk mencegah serangan (SQL Injection, XSS, dll.).
Rate Limiting: Terapkan rate limiting pada endpoint otentikasi untuk mencegah brute-force attack.
Role-Based Access Control (RBAC): Untuk MVP, ini disederhanakan (satu pengguna utama per restoran). Di masa depan, akan diperluas untuk multi-user internal.
Multi-Tenant Isolation: Pastikan Backend API secara ketat mengisolasi data antar tenant dengan selalu memvalidasi restaurant_id dari token dan mengarahkan ke database/skema yang tepat.

5.2 Skalabilitas
Backend: Dirancang sebagai stateless microservices/modular monolith. Dapat diskalakan secara horizontal (menambahkan lebih banyak instance server).
Database:
Central DB: Jika jumlah restoran sangat besar, bisa di-scale secara vertikal atau sharding berdasarkan range ID.
Restaurant-Specific DBs: Model database/skema per tenant secara inheren skalabel dalam hal isolasi beban kerja. Jika satu tenant memiliki traffic tinggi, itu tidak langsung memengaruhi tenant lain secara signifikan pada tingkat database. Skalabilitas horizontal dapat dicapai dengan menempatkan tenant pada instance database yang berbeda.
Caching: Terapkan caching di Backend untuk data yang sering diakses dan jarang berubah (misalnya, daftar kategori menu), untuk mengurangi beban database.
Load Balancer: Gunakan load balancer di depan Backend API untuk mendistribusikan traffic.

5.3 Ketersediaan & Keandalan
Monitoring: Implementasikan monitoring untuk backend API, database, dan infrastruktur untuk mendeteksi masalah secara proaktif.
Logging: Catat semua event penting di backend untuk debugging dan audit.
Backup & Restore: Terapkan strategi backup reguler untuk Central DB dan semua Restaurant-Specific DBs. Pastikan kemampuan restore.
Redundansi: Server dan database di-hosting di lingkungan yang redundan untuk mencegah single point of failure.

5.4 Performa
Optimasi Query: Pastikan semua query database dioptimalkan dengan indeks yang tepat.
Connection Pooling: Gunakan connection pooling di Backend untuk mengelola koneksi database secara efisien.
Efisiensi Algoritma: Tulis kode yang efisien, terutama untuk alur transaksi penjualan yang melibatkan pengurangan stok.

### 6. Deployment Strategy (Rekomendasi Awal)
Hosting Backend & Databases: Cloud provider (AWS, Google Cloud, Azure, DigitalOcean, Heroku) untuk kemudahan manajemen, skalabilitas, dan ketersediaan.
Contoh: VM untuk Backend API, Managed Database Service (misal: AWS RDS PostgreSQL) untuk Central DB dan Restaurant-Specific DBs.
CI/CD: Implementasikan Continuous Integration/Continuous Deployment (CI/CD) pipeline untuk otomatisasi pengujian, build, dan deployment kode.
Containerization (Opsional untuk MVP): Gunakan Docker untuk mengemas Backend API dan database. Dapat di-deploy ke Kubernetes atau Docker Swarm di masa depan.

### 7. Rencana Pengujian
Unit Tests: Untuk logika bisnis di Backend dan state management di Frontend.
Integration Tests: Menguji interaksi antar komponen (misal: Backend API dengan database).
API Tests: Menguji endpoint API secara independen.
UI/E2E Tests: Menguji alur pengguna dari Frontend (opsional untuk MVP yang sangat ramping).
Manual Testing: Pengujian fungsionalitas di perangkat fisik.
Performance Testing: Menguji beban kerja sistem pada jumlah pengguna/transaksi tertentu.
Security Testing: Penetration testing dasar (opsional untuk MVP).

### 8. Rencana Pengembangan Selanjutnya (Pasca-MVP)

(Bagian ini akan sama dengan SRS sebelumnya, fokus pada fitur-fitur yang akan ditambahkan setelah MVP berhasil divalidasi dan di-deploy).
