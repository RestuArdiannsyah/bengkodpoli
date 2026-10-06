# Poliklinik App

Aplikasi manajemen klinik berbasis web yang dibangun dengan Laravel 13. Mendukung tiga jenis pengguna: Admin, Dokter, dan Pasien, masing-masing dengan dashboard dan fitur tersendiri.

---

## Daftar Isi

- [Fitur](#fitur)
- [Tech Stack](#tech-stack)
- [Persyaratan Sistem](#persyaratan-sistem)
- [Instalasi](#instalasi)
- [Akun Default](#akun-default)
- [Struktur Role dan Akses](#struktur-role-dan-akses)
- [Struktur Database](#struktur-database)
- [Struktur Folder](#struktur-folder)
- [Konfigurasi MySQL](#konfigurasi-mysql)
- [Menjalankan Aplikasi](#menjalankan-aplikasi)
- [Tools Tambahan](#tools-tambahan)
- [Catatan Pengembangan](#catatan-pengembangan)

---

## Fitur

| Fitur | Admin | Dokter | Pasien |
|-------|:-----:|:------:|:------:|
| Dashboard | Ya | Ya | Ya |
| Manajemen Poli | Dalam pengembangan | - | - |
| Manajemen Dokter | Dalam pengembangan | - | - |
| Manajemen Pasien | Dalam pengembangan | - | - |
| Manajemen Obat | Dalam pengembangan | - | - |
| Jadwal Periksa | - | Ya | - |
| Periksa Pasien | - | Ya | - |
| Riwayat Pasien | - | Ya | - |
| Pendaftaran Periksa | - | - | Ya |
| Registrasi Mandiri | - | - | Ya |

Fitur yang sudah berjalan:

- Login dan logout dengan redirect otomatis berdasarkan role
- Registrasi pasien mandiri dengan generate nomor rekam medis otomatis (format: `YYYYMM-NNN`)
- Role-based access control menggunakan middleware custom
- Tiga dashboard terpisah untuk Admin, Dokter, dan Pasien
- Sidebar dinamis yang menyesuaikan menu berdasarkan role aktif

---

## Tech Stack

**Backend**

| Package | Versi | Keterangan |
|---------|-------|------------|
| PHP | ^8.3 | Bahasa pemrograman utama |
| Laravel | ^13.17 | Framework aplikasi |
| laravel/tinker | ^3.0 | REPL interaktif untuk debug |

**Frontend**

| Package | Versi | Keterangan |
|---------|-------|------------|
| Tailwind CSS | ^4.3.3 | Utility-first CSS framework |
| DaisyUI | ^5.7.47 | Komponen UI berbasis Tailwind |
| Vite | ^8.0.0 | Build tool dan dev server |
| Font Awesome 7 | CDN | Library ikon |
| Plus Jakarta Sans | CDN | Font utama (Google Fonts) |

**Development Tools**

| Package | Keterangan |
|---------|------------|
| laravel/pint | PHP code style fixer |
| laravel/pail | Real-time log viewer di terminal |
| fakerphp/faker | Generator data dummy |
| phpunit/phpunit | Framework unit testing |
| nunomaduro/collision | Error display yang lebih informatif di CLI |

**Database**

SQLite digunakan secara default dan sudah dikonfigurasi tanpa perlu setup tambahan. Aplikasi juga dapat diubah ke MySQL atau PostgreSQL melalui file `.env`.

---

## Persyaratan Sistem

Pastikan tools berikut sudah terinstall sebelum memulai:

- PHP >= 8.3
- Composer >= 2.x
- Node.js >= 18.x dan npm >= 9.x
- Git

Verifikasi dengan menjalankan perintah berikut:

```bash
php -v
composer -V
node -v
npm -v
```

---

## Instalasi

### 1. Clone Repository

```bash
git clone https://github.com/username/poliklinik-app.git
cd poliklinik-app
```

Ganti `username` dengan username GitHub yang sesuai.

---

### 2. Install Dependency PHP

```bash
composer install
```

---

### 3. Buat File Konfigurasi Environment

Salin file contoh konfigurasi:

```bash
cp .env.example .env
```

Generate application key:

```bash
php artisan key:generate
```

---

### 4. Siapkan Database

Aplikasi menggunakan SQLite secara default. Buat file database-nya terlebih dahulu:

**Linux / macOS:**
```bash
touch database/database.sqlite
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType File -Path database/database.sqlite
```

Jalankan migrasi untuk membuat semua tabel:

```bash
php artisan migrate
```

Isi data awal termasuk akun default:

```bash
php artisan db:seed
```

Atau jalankan migrasi dan seeder sekaligus:

```bash
php artisan migrate --seed
```

---

### 5. Install Dependency Node.js

```bash
npm install
```

---

### 6. Build Asset Frontend

Untuk production:

```bash
npm run build
```

Untuk development dengan hot reload (jalankan di terminal terpisah):

```bash
npm run dev
```

---

### 7. Jalankan Server

```bash
php artisan serve
```

Akses aplikasi di browser: **http://localhost:8000**

---

### Shortcut Setup Lengkap

Composer menyediakan script yang menggabungkan seluruh proses di atas:

```bash
# Setup penuh: install, .env, key:generate, migrate, npm install, build
composer run setup

# Jalankan PHP server dan Vite bersamaan
composer run dev
```

---

## Akun Default

Setelah menjalankan seeder, tiga akun berikut tersedia untuk keperluan testing:

| Role | Email | Password |
|------|-------|----------|
| Admin | admin@gmail.com | admin |
| Dokter | dokter@gmail.com | dokter |
| Pasien | pasien@gmail.com | pasien |

> **Perhatian:** Ganti semua password di atas sebelum mendeploy ke lingkungan production.

Registrasi mandiri hanya tersedia untuk role `pasien`. Akun Admin dan Dokter harus dibuat melalui seeder atau langsung via Tinker.

---

## Struktur Role dan Akses

Setiap role memiliki URL prefix tersendiri yang dijaga oleh middleware `auth` dan `role`:

```
/admin/*     ->  hanya dapat diakses oleh role: admin
/dokter/*    ->  hanya dapat diakses oleh role: dokter
/pasien/*    ->  hanya dapat diakses oleh role: pasien
```

Setelah login berhasil, pengguna otomatis diarahkan ke dashboard sesuai role:

```
admin   ->  /admin/dashboard
dokter  ->  /dokter/dashboard
pasien  ->  /pasien/dashboard
```

Jika pengguna mencoba mengakses URL di luar role-nya, aplikasi akan merespons dengan **403 Unauthorized**.

---

## Struktur Database

### Tabel `users`

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| id | bigint | Primary key |
| nama | varchar | Nama lengkap |
| email | varchar | Unik |
| password | varchar | Hash bcrypt |
| role | enum | `admin`, `dokter`, atau `pasien` |
| no_ktp | varchar | Nomor KTP, unik |
| no_hp | varchar | Nomor handphone |
| alamat | text | Alamat lengkap |
| no_rm | varchar(25) | Nomor rekam medis, digenerate otomatis |
| id_poli | bigint FK | Poli tempat dokter bertugas (nullable) |

### Tabel `poli`

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| id | bigint | Primary key |
| nama_poli | varchar(25) | Nama spesialisasi |
| keterangan | text | Deskripsi tambahan (nullable) |

### Tabel `jadwal_periksa`

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| id | bigint | Primary key |
| id_dokter | bigint FK | Referensi ke `users` |
| hari | enum | Senin hingga Minggu |
| jam_mulai | time | Jam mulai praktik |
| jam_selesai | time | Jam selesai praktik |

### Tabel `daftar_poli`

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| id | bigint | Primary key |
| id_pasien | bigint FK | Referensi ke `users` |
| id_jadwal | bigint FK | Referensi ke `jadwal_periksa` |
| keluhan | text | Keluhan utama pasien |
| no_antrian | int | Nomor antrian |

### Tabel `periksa`

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| id | bigint | Primary key |
| id_daftar_poli | bigint FK | Referensi ke `daftar_poli` |
| tgl_periksa | datetime | Tanggal dan waktu periksa |
| catatan | text | Catatan dokter (nullable) |
| biaya_periksa | int | Total biaya dalam Rupiah |

### Tabel `obat`

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| id | bigint | Primary key |
| nama_obat | varchar | Nama obat |
| kemasan | varchar(35) | Jenis kemasan (nullable) |
| harga | int | Harga satuan dalam Rupiah |

### Tabel `detail_periksa`

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| id | bigint | Primary key |
| id_periksa | bigint FK | Referensi ke `periksa` |
| id_obat | bigint FK | Referensi ke `obat` |

### Relasi Antar Tabel

```
users (dokter)  --< jadwal_periksa
users (pasien)  --< daftar_poli
poli            --< users (dokter)
jadwal_periksa  --< daftar_poli
daftar_poli     --< periksa
periksa         --< detail_periksa
obat            --< detail_periksa
```

---

## Struktur Folder

```
poliklinik-app/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── AuthController.php        # Login, register, logout
│   │   └── Middleware/
│   │       └── RoleMiddleware.php         # Validasi role pengguna
│   ├── Models/
│   │   ├── User.php
│   │   ├── Poli.php
│   │   ├── JadwalPeriksa.php
│   │   ├── DaftarPoli.php
│   │   ├── Periksa.php
│   │   ├── DetailPeriksa.php
│   │   └── Obat.php
│   └── Providers/
│       └── AppServiceProvider.php
│
├── database/
│   ├── migrations/                        # Semua file migrasi tabel
│   ├── seeders/
│   │   ├── DatabaseSeeder.php
│   │   └── UserSeeder.php                # Akun default tiga role
│   └── database.sqlite                   # File database SQLite
│
├── resources/
│   ├── css/
│   │   └── app.css                       # Konfigurasi Tailwind v4 dan DaisyUI
│   ├── js/
│   │   └── app.js
│   └── views/
│       ├── auth/
│       │   ├── login.blade.php
│       │   └── register.blade.php
│       ├── admin/
│       │   └── dashboard.blade.php
│       ├── dokter/                        # Dalam pengembangan
│       ├── pasien/                        # Dalam pengembangan
│       └── components/
│           ├── layouts/
│           │   ├── app.blade.php          # Layout utama (authenticated)
│           │   └── guest.blade.php        # Layout tamu (login/register)
│           └── partials/
│               ├── sidebar.blade.php
│               ├── header.blade.php
│               └── footer.blade.php
│
├── routes/
│   └── web.php                            # Definisi seluruh route aplikasi
│
├── public/
│   ├── images/
│   │   └── logo-bengkot.png
│   └── build/                             # Output asset dari Vite
│
├── .env.example                           # Template konfigurasi environment
├── composer.json
└── package.json
```

---

## Konfigurasi MySQL

Jika ingin menggunakan MySQL sebagai pengganti SQLite (misalnya dengan Laragon atau XAMPP):

1. Buat database baru di MySQL, contoh: `poliklinik_db`

2. Edit file `.env` dan sesuaikan bagian berikut:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=poliklinik_db
DB_USERNAME=root
DB_PASSWORD=
```

3. Pastikan baris `DB_CONNECTION=sqlite` sudah dihapus atau dikomentari.

4. Jalankan ulang migrasi:

```bash
php artisan migrate --seed
```

---

## Menjalankan Aplikasi

### Mode Development

Buka dua terminal secara bersamaan.

**Terminal 1 — PHP Server:**
```bash
php artisan serve
```

**Terminal 2 — Vite (hot reload):**
```bash
npm run dev
```

Akses di: **http://localhost:8000**

---

### Mode Production

```bash
npm run build
php artisan serve
```

---

## Tools Tambahan

**Melihat log secara real-time:**
```bash
php artisan pail
```

**Laravel Tinker — REPL interaktif:**
```bash
php artisan tinker
```

Contoh penggunaan Tinker untuk mengecek data user:
```php
>>> User::all(['nama', 'email', 'role'])
```

**Format kode PHP secara otomatis:**
```bash
./vendor/bin/pint
```

**Menjalankan test:**
```bash
php artisan test
```

---

## Catatan Pengembangan

Beberapa hal yang perlu diperhatikan saat melanjutkan pengembangan proyek ini:

1. **View yang belum dibuat** — `dokter/dashboard.blade.php` dan `pasien/dashboard.blade.php` belum ada. Login sebagai dokter atau pasien akan menghasilkan error `View not found` hingga file tersebut dibuat.

2. **Route yang belum didefinisikan** — Beberapa link di sidebar (Jadwal Periksa, Periksa Pasien, Riwayat Pasien, Pendaftaran Periksa) belum terdaftar di `routes/web.php`. Mengklik link tersebut akan menyebabkan error route not found.

3. **Bug pada model `DaftarPoli`** — Relasi `belongsTo` merujuk ke `Pasien::class` yang tidak ada. Seharusnya menggunakan `User::class`.

4. **Field nama pengguna** — Field nama di database adalah `nama`, bukan `name` (default Laravel). Perhatikan hal ini saat menampilkan nama pengguna di view agar tidak terjadi `undefined property`.

5. **Keamanan production** — Sebelum deploy, ubah `APP_DEBUG=false` dan `APP_ENV=production` di file `.env`, serta ganti seluruh password akun default.

---

## Lisensi

Proyek ini menggunakan lisensi [MIT](https://opensource.org/licenses/MIT).
