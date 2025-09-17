## Entity-Relationship Diagram (ERD) - Aplikasi Manajemen Rumah Makan

- **Versi**: 1.0
- **Tanggal**: 17 September 2025
- **Penyusun**: Muhamad Azkal Fahmi Yahya_221240001336 dan Khaf Sari Inayati_221240001228

### Konvensi
- **Entitas (Tabel)**: Persegi panjang
- **Atribut (Kolom)**: Oval, dengan Primary Key (PK) digarisbawahi. Foreign Key (FK) ditandai.
- **Relasi**: Garis yang menghubungkan entitas, dengan notasi Crow's Foot untuk menunjukkan kardinalitas (satu-ke-satu, satu-ke-banyak, banyak-ke-banyak).

### 1) ERD untuk Database Pusat (Backend)
- **Tujuan**: Mengelola informasi pendaftaran semua restoran, kredensial pengguna utama, dan status langganan. Database ini akan menjadi titik masuk utama bagi aplikasi Flutter untuk otentikasi dan identifikasi database SQL lokal mana yang harus diakses.

```mermaid
erDiagram
    USERS {
        UUID id PK "UUID unik pengguna"
        VARCHAR email UNIQUE "Email pengguna"
        VARCHAR password_hash "Hash password pengguna"
        VARCHAR name "Nama pengguna"
        BOOLEAN is_super_admin "Apakah pengguna adalah Super Admin"
        DATETIME created_at "Timestamp pembuatan"
        DATETIME updated_at "Timestamp pembaruan"
    }

    RESTAURANTS {
        UUID id PK "UUID unik restoran"
        VARCHAR restaurant_name "Nama restoran"
        UUID primary_user_id FK "ID pengguna utama yang mendaftar"
        VARCHAR db_connection_string UNIQUE "String koneksi ke DB SQL lokal restoran"
        VARCHAR subscription_status "Status langganan (trial, active, expired, cancelled)"
        DATETIME trial_ends_at "Tanggal berakhir masa trial"
        DATETIME subscription_ends_at "Tanggal berakhir langganan"
        VARCHAR address "Alamat restoran"
        VARCHAR phone "Nomor telepon restoran"
        VARCHAR currency_symbol "Simbol mata uang (default 'Rp')"
        DATETIME created_at "Timestamp pembuatan"
        DATETIME updated_at "Timestamp pembaruan"
    }

    USERS ||--o{ RESTAURANTS : "mendaftar"
```

#### Penjelasan Atribut Database Pusat
- **USERS**
  - id: Primary key, ID unik untuk setiap pengguna (bisa UUID).
  - email: Email pengguna, digunakan untuk login, harus unik.
  - password_hash: Hash dari password pengguna untuk keamanan.
  - name: Nama lengkap pengguna.
  - is_super_admin: Bendera untuk mengidentifikasi Super Admin.
  - created_at, updated_at: Timestamp standar.
- **RESTAURANTS**
  - id: Primary key, ID unik untuk setiap restoran yang terdaftar (bisa UUID).
  - restaurant_name: Nama restoran.
  - primary_user_id: Foreign key ke USERS.id, menunjukkan siapa yang mendaftarkan/mengelola restoran ini.
  - db_connection_string: Kunci utama arsitektur multi-tenant; menyimpan string koneksi unik atau ID mapping.
  - subscription_status: Status langganan.
  - trial_ends_at, subscription_ends_at: Untuk manajemen langganan.
  - address, phone, currency_symbol: Informasi dasar restoran.
  - created_at, updated_at: Timestamp standar.

### 2) ERD Template untuk Database SQL Lokal Per Restoran
- **Tujuan**: Skema standar untuk setiap restoran pelanggan. Menyimpan data operasional (menu, inventaris, penjualan, resep).

```mermaid
erDiagram
    MENU_CATEGORIES {
        UUID id PK "UUID unik kategori"
        VARCHAR name UNIQUE "Nama kategori"
        TEXT description "Deskripsi kategori (opsional)"
        DATETIME created_at
        DATETIME updated_at
    }

    MENU_ITEMS {
        UUID id PK "UUID unik item menu"
        VARCHAR name UNIQUE "Nama item menu"
        TEXT description "Deskripsi item (opsional)"
        DECIMAL(10,2) price "Harga jual item"
        UUID category_id FK "ID kategori menu"
        VARCHAR image_url "URL gambar item (opsional)"
        BOOLEAN is_available "Status ketersediaan item"
        DATETIME created_at
        DATETIME updated_at
    }

    INGREDIENTS {
        UUID id PK "UUID unik bahan baku"
        VARCHAR name UNIQUE "Nama bahan baku"
        VARCHAR sku UNIQUE "SKU bahan baku (opsional)"
        VARCHAR unit_of_measure "Satuan ukuran (gram, pcs, ml)"
        DECIMAL(10,2) purchase_price_per_unit "Harga beli/modal per satuan"
        DECIMAL(10,2) current_stock "Jumlah stok saat ini"
        DATETIME created_at
        DATETIME updated_at
    }

    RECIPE_DETAILS {
        UUID id PK "UUID unik detail resep"
        UUID menu_item_id FK "ID item menu"
        UUID ingredient_id FK "ID bahan baku"
        DECIMAL(10,2) quantity_needed "Kuantitas bahan baku yang dibutuhkan per porsi"
        DATETIME created_at
        DATETIME updated_at
    }

    SALES_TRANSACTIONS {
        UUID id PK "UUID unik transaksi penjualan"
        DATETIME transaction_date "Tanggal dan waktu transaksi"
        DECIMAL(10,2) total_amount_paid "Total uang yang dibayarkan pelanggan"
        DECIMAL(10,2) cash_received "Uang tunai yang diterima dari pelanggan"
        DECIMAL(10,2) change_amount "Jumlah kembalian"
        UUID processed_by_user_id "ID user utama yang memproses transaksi"
        DATETIME created_at
        DATETIME updated_at
    }

    SALES_TRANSACTION_DETAILS {
        UUID id PK "UUID unik detail transaksi"
        UUID transaction_id FK "ID transaksi penjualan"
        UUID menu_item_id FK "ID item menu yang terjual"
        VARCHAR menu_item_name_at_sale "Nama item menu saat penjualan"
        INTEGER quantity_sold "Kuantitas item yang terjual"
        DECIMAL(10,2) price_at_sale "Harga jual item saat transaksi"
        DECIMAL(10,2) cost_price_total_for_items "Total HPP untuk item ini (opsional)"
        DECIMAL(10,2) subtotal_revenue "Subtotal pendapatan untuk item ini"
        DATETIME created_at
        DATETIME updated_at
    }

    MENU_CATEGORIES ||--o{ MENU_ITEMS : "memiliki"
    MENU_ITEMS ||--o{ RECIPE_DETAILS : "memiliki resep"
    INGREDIENTS ||--o{ RECIPE_DETAILS : "digunakan dalam"
    SALES_TRANSACTIONS ||--o{ SALES_TRANSACTION_DETAILS : "terdiri dari"
    MENU_ITEMS ||--o{ SALES_TRANSACTION_DETAILS : "terjual sebagai"
```

#### Penjelasan Atribut Database SQL Lokal Per Restoran
- **MENU_CATEGORIES**: Menyimpan kategori menu (misal: "Appetizer", "Main Course").
  - id: Primary key, UUID unik.
  - name: Nama kategori, harus unik dalam satu restoran.
- **MENU_ITEMS**: Menyimpan detail setiap item menu.
  - id: Primary key, UUID unik.
  - name: Nama item menu, harus unik dalam satu restoran.
  - price: Harga jual item.
  - category_id: Foreign key ke MENU_CATEGORIES.id.
  - image_url: URL gambar item, jika disimpan di penyimpanan terpisah.
  - is_available: Status ketersediaan.
- **INGREDIENTS**: Menyimpan daftar bahan baku.
  - id: Primary key, UUID unik.
  - name: Nama bahan baku, harus unik dalam satu restoran.
  - sku: Opsional, Stock Keeping Unit.
  - unit_of_measure: Satuan ukuran (misal: gram, ml, pcs).
  - purchase_price_per_unit: Harga modal per satuan.
  - current_stock: Jumlah stok saat ini.
- **RECIPE_DETAILS**: Menghubungkan item menu dengan bahan baku dan kuantitasnya.
  - id: Primary key, UUID unik.
  - menu_item_id: Foreign key ke MENU_ITEMS.id.
  - ingredient_id: Foreign key ke INGREDIENTS.id.
  - quantity_needed: Berapa banyak bahan baku yang dibutuhkan untuk 1 porsi menu.
- **SALES_TRANSACTIONS**: Menyimpan data utama setiap transaksi penjualan.
  - id: Primary key, UUID unik.
  - transaction_date: Tanggal dan waktu transaksi.
  - total_amount_paid: Total yang dibayar pelanggan.
  - cash_received: Uang tunai yang diberikan.
  - change_amount: Kembalian.
  - processed_by_user_id: ID pengguna utama dari database pusat yang melakukan transaksi (untuk pelacakan). Penting di MVP karena satu pengguna utama per restoran.
- **SALES_TRANSACTION_DETAILS**: Menyimpan detail item-item yang terjual dalam suatu transaksi.
  - id: Primary key, UUID unik.
  - transaction_id: Foreign key ke SALES_TRANSACTIONS.id.
  - menu_item_id: Foreign key ke MENU_ITEMS.id (referensi ke data menu saat ini).
  - menu_item_name_at_sale: Menyimpan nama item menu saat transaksi terjadi.
  - quantity_sold: Jumlah item yang terjual.
  - price_at_sale: Harga jual item saat transaksi.
  - cost_price_total_for_items: Total HPP untuk item ini dalam transaksi.

  - subtotal_revenue: quantity_sold * price_at_sale.
