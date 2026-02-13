# Panduan Instalasi MovieBox di VPS

Panduan ini menjelaskan langkah-langkah instalasi aplikasi MovieBox dari awal hingga berjalan di VPS (Virtual Private Server), termasuk cara membuat akun admin Filament.

## 📋 Persyaratan Server (Requirements)

Pastikan VPS Anda sudah terinstall:

- **OS**: Ubuntu 22.04 LTS / 24.04 LTS (Recommended)
- **Web Server**: Nginx (Recommended) atau Apache
- **PHP**: Versi 8.2 atau lebih baru
    - Extensions: `bcmath`, `ctype`, `curl`, `dom`, `fileinfo`, `mbstring`, `pdo`, `pdo_mysql`, `tokenizer`, `xml`, `intl`, `zip`
- **Database**: MySQL 8.0+ atau MariaDB 10.6+
- **Composer**: Versi 2.x
- **Node.js & NPM**: Versi 18.x atau 20.x

---

## 🚀 Langkah-langkah Instalasi

### 1. Clone Repository & Setup Folder

Masuk ke folder `/var/www` (atau direktori root web Anda):

```bash
cd /var/www
git clone https://github.com/username/moviebox.git
cd moviebox
```

### 2. Setup Environment Variables

Salin file `.env.example` menjadi `.env` dan sesuaikan konfigurasinya:

```bash
cp .env.example .env
nano .env
```

**Konfigurasi Penting di `.env`:**

```ini
APP_NAME=MovieBox
APP_ENV=production
APP_KEY=
APP_DEBUG=false
APP_URL=https://domain-anda.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nama_database
DB_USERNAME=user_database
DB_PASSWORD=password_database
```

### 3. Install Dependencies PHP (Composer)

Jalankan perintah ini untuk menginstall library PHP:

```bash
composer install --optimize-autoloader --no-dev
```

### 4. Setup Key & Permission

Generate application key dan atur permission folder:

```bash
php artisan key:generate
php artisan storage:link

# Set permission folder
chown -R www-data:www-data /var/www/moviebox
chmod -R 775 storage bootstrap/cache
```

### 5. Install Dependencies Frontend (Node.js)

Install library JavaScript dan build aset (CSS/JS):

```bash
npm install
npm run build
```

### 6. Migrasi Database

Pastikan database sudah dibuat di MySQL, lalu jalankan migrasi:

```bash
php artisan migrate --force
```

---

## 👤 Membuat Akun Admin (Filament)

Untuk mengakses halaman admin panel, Anda perlu membuat user Filament:

```bash
php artisan make:filament-user
```

Ikuti prompt yang muncul:

1. **Name**: (Masukkan nama Anda)
2. **Email**: (Masukkan email login, misal: admin@moviebox.com)
3. **Password**: (Masukkan password kuat)

Setelah sukses, Anda bisa login di: `https://domain-anda.com/admin`

---

## 🌐 Konfigurasi Web Server (Contoh Nginx)

Buat file konfigurasi Nginx: `/etc/nginx/sites-available/moviebox`

```nginx
server {
    listen 80;
    server_name domain-anda.com;
    root /var/www/moviebox/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock; # Sesuaikan versi PHP
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Aktifkan site dan restart Nginx:

```bash
ln -s /etc/nginx/sites-available/moviebox /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

---

## 📂 Struktur File & Folder Penting

Berikut adalah struktur file MovieBox untuk referensi bagian mana yang aman diedit:

```text
moviebox/
├── app/
│   ├── Filament/                # 🔧 Admin Panel Logic
│   │   └── Resources/           # Controllers untuk menu Admin (MovieResource, etc)
│   ├── Http/
│   │   └── Controllers/         # 🎮 Main Controllers (Frontend)
│   │       ├── HomeController.php   # Logic halaman utama & override Title
│   │       └── ...
│   ├── Services/
│   │   └── MovieService.php     # 🔌 API Integration Logic (Fetch data dari API external)
│   └── Models/                  # 📦 Database Models
├── config/                      # ⚙️ Konfigurasi Aplikasi
├── database/                    # 🗄️ Migrations & Seeds
├── public/                      # 🌐 Public Access Directories
│   ├── css/
│   │   └── style.css            # 🎨 FILE CSS UTAMA (Edit style di sini)
│   ├── js/
│   │   └── app.js               # 📜 FILE JS UTAMA (Logic frontend, toggle menu, dll)
│   └── ...
├── resources/
│   └── views/                   # 👁️ Views / Template HTML (Blade)
│       ├── layouts/
│       │   └── app.blade.php    # 🏗️ Layout Utama (Header, Footer, Mobile Drawer HTML)
│       ├── partials/            # 🧩 Potongan kode (bisa diinclude)
│       ├── home.blade.php       # 🏠 Halaman Home
│       ├── movies.blade.php     # 🎬 Halaman List Movies & Filter
│       ├── series.blade.php     # 📺 Halaman List Series & Filter
│       └── watch.blade.php      # 🎥 Halaman Nonton/Player
├── routes/
│   └── web.php                  # 🛣️ Definisi URL/Route
└── .env                         # 🔐 Environment Variables (Database, API URL, Debug mode)
```

### 📝 Catatan Edit File:

- **CSS**: Selalu edit `public/css/style.css`, hindari inline CSS di blade.
- **JavaScript**: Edit `public/js/app.js` untuk logic global.
- **Title Halaman Home**: Edit `app/Http/Controllers/HomeController.php` di bagian `$titleOverrides`.
- **API External**: Jika ada perubahan endpoint, cek `app/Services/MovieService.php`.

---

## 🛠️ Optimasi (Production)

Setelah semua berjalan normal, jalankan perintah ini untuk performa maksimal:

```bash
php artisan optimize:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

Jika ada update kode (git pull), jangan lupa jalankan ulang perintah optimasi di atas dan `npm run build` jika ada perubahan CSS/JS.
