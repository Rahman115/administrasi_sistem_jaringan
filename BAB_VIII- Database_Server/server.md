

---

⚙️ 1. Konsep Singkat (Biar Tidak Sekadar Hafal)

MariaDB di Debian bekerja sebagai:

Service (daemon) → berjalan di background

Listener → default port 3306

DBMS → mengelola database, tabel, user


Struktur sederhananya:

Client → TCP/IP → MariaDB Server → Storage (Disk)


---

🧱 2. Instalasi MariaDB di Debian

Update Repository

Karena Debian suka basi kalau tidak disegarkan:

sudo apt update && sudo apt upgrade -y

Install MariaDB Server

sudo apt install mariadb-server -y

Cek Status Service

sudo systemctl status mariadb

Kalau hidup:

active (running)

Kalau mati… ya, selamat datang di dunia troubleshooting.


---

🔐 3. Secure Installation (WAJIB, jangan malas)

MariaDB menyediakan wizard keamanan:

sudo mysql_secure_installation

Ikuti langkah:

1. Set root password → YES


2. Remove anonymous users → YES


3. Disallow root login remotely → YES


4. Remove test database → YES


5. Reload privilege → YES



Ini bukan formalitas. Ini garis tipis antara “server” dan “undangan hacker”.


---

🧠 4. Masuk ke MariaDB

sudo mysql -u root -p

Kalau berhasil:

MariaDB [(none)]>

Kau sekarang punya kekuasaan kecil atas data.


---

🏗️ 5. Membuat Database & User

Buat Database

CREATE DATABASE sekolah;

Buat User

CREATE USER 'admin'@'localhost' IDENTIFIED BY 'passwordku';

Beri Hak Akses

GRANT ALL PRIVILEGES ON sekolah.* TO 'admin'@'localhost';
FLUSH PRIVILEGES;

Kalau lupa FLUSH, sistemmu akan pura-pura tidak dengar.


---

🌐 6. Konfigurasi Akses Remote (Opsional tapi sering dicari)

Edit file config:

sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

Cari:

bind-address = 127.0.0.1

Ubah jadi:

bind-address = 0.0.0.0

Artinya:

> “Silakan masuk… tapi tetap pakai password, jangan liar.”




---

Restart Service

sudo systemctl restart mariadb


---

Izinkan User Remote

CREATE USER 'admin'@'%' IDENTIFIED BY 'passwordku';
GRANT ALL PRIVILEGES ON sekolah.* TO 'admin'@'%';
FLUSH PRIVILEGES;


---

🔥 7. Buka Firewall (Kalau pakai UFW)

sudo ufw allow 3306

Kalau lupa ini, servermu akan seperti rumah dengan pintu terkunci dari dalam.


---

📁 8. Struktur File Penting

Config utama:


/etc/mysql/

Data database:


/var/lib/mysql/

Log:


/var/log/mysql/

Jangan sembarangan sentuh /var/lib/mysql/ kecuali kau suka hidup berisiko.


---

⚡ 9. Pengujian Koneksi

Dari client:

mysql -u admin -p -h IP_SERVER

Kalau gagal:

cek firewall

cek bind-address

cek user permission


Biasanya bukan sistem yang salah. Biasanya… manusia.


---

🧩 10. Gambaran Mekanisme Setelah Terpasang

1. Service MariaDB aktif


2. Port 3306 terbuka


3. Client kirim query


4. Server validasi user


5. Query dieksekusi


6. Data diambil dari disk


7. Hasil dikirim balik



Sederhana. Tapi kalau satu bagian gagal, semuanya ikut tenggelam.


---

🧠 Catatan Realistis

Jangan pakai root untuk aplikasi

Backup itu bukan opsional

Jangan buka akses ke internet tanpa pembatasan


Karena:

> database bukan tempat menyimpan data saja… tapi juga menyimpan kesalahan manusia.


