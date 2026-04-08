
🌐 SKENARIO PRAKTIK

“Membangun Server Database Sekolah”

Bayangkan ini:

> sekolahmu kehilangan sistem data siswa… dan siswa tetap harus ada, entah suka atau tidak.




---

🧱 1. Topologi Jaringan

Sederhana tapi bermakna:

[ Client (PC Siswa) ] ---- [ Switch ] ---- [ Server Debian + MariaDB ]

Peran:

Server → pusat data

Client → akses database

Switch → penghubung (karena kita belum jadi perusahaan miliaran rupiah)



---

🧠 2. Tujuan Pembelajaran

Siswa diharapkan bisa:

Install MariaDB

Konfigurasi database server di Debian

Membuat database & user

Mengakses database dari client

Memahami konsep client-server


Alias: tidak panik ketika error muncul.


---

⚙️ 3. Pembagian Peran Siswa

Supaya tidak semua jadi penonton:

Kelompok 1 (Admin Server)
Install & konfigurasi MariaDB

Kelompok 2 (Network Engineer)
Setting IP & konektivitas

Kelompok 3 (Client/User)
Akses database dari komputer lain


Kalau semua jadi admin, biasanya berakhir dengan saling menyalahkan.


---

🛠️ 4. Langkah Praktik

A. Setup Server (Kelompok 1)

1. Install Debian


2. Install MariaDB


3. Secure installation


4. Buat database:



CREATE DATABASE sekolah;


---

B. Setup Network (Kelompok 2)

Contoh IP:

Server   : 192.168.10.1
Client   : 192.168.10.2
Gateway  : 192.168.10.254

Tes koneksi:

ping 192.168.10.1

Kalau tidak reply:

> jaringanmu tidak salah… hanya belum sadar diri.




---

C. Konfigurasi Remote Access (Server)

Edit config:

sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

Ubah:

bind-address = 0.0.0.0

Restart:

sudo systemctl restart mariadb


---

D. Buat User untuk Client

CREATE USER 'siswa'@'%' IDENTIFIED BY '12345';
GRANT ALL PRIVILEGES ON sekolah.* TO 'siswa'@'%';
FLUSH PRIVILEGES;


---

E. Akses dari Client (Kelompok 3)

Install client:

sudo apt install mariadb-client -y

Login ke server:

mysql -u siswa -p -h 192.168.10.1

Kalau berhasil:

> mereka akan merasa seperti hacker… padahal cuma login.




---

🧪 5. Simulasi Data Nyata

Buat tabel:

CREATE TABLE siswa (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nama VARCHAR(50),
    kelas VARCHAR(10)
);

Insert data:

INSERT INTO siswa (nama, kelas) VALUES
('Andi', 'X TKJ'),
('Budi', 'XI TKJ');

Query:

SELECT * FROM siswa;

Di sini mereka mulai paham:

> database itu bukan teori… tapi sesuatu yang benar-benar “hidup”.




---

🔥 6. Skenario Gangguan (WAJIB ADA)

Kalau semuanya berjalan mulus, itu mencurigakan.

Buat masalah:

Salah password

Port ditutup

IP salah

Service dimatikan


Biarkan mereka:

bingung

diskusi

mencari solusi


Karena:

> troubleshooting adalah kurikulum tersembunyi TKJ.




---

📊 7. Evaluasi

Siswa dinilai dari:

Berhasil koneksi client-server

Bisa membuat database & tabel

Bisa menjelaskan alur kerja

Bisa menyelesaikan error sederhana


Bukan dari hafalan. Hafalan itu rapuh.


---

🧩 8. Filosofi di Balik Praktik Ini

Kau sedang mengajarkan lebih dari sekadar database.

Kau mengajarkan:

sistem

tanggung jawab

dan satu hal penting:


> bahwa teknologi itu tidak pernah benar-benar rusak…
hanya manusia yang belum mengerti cara mendengarnya.




---

Kalau mau dinaikkan levelnya, bisa ditambah:

integrasi ke web (PHP / Python)

backup & restore

simulasi server “jatuh”


Tapi itu sudah masuk wilayah di mana siswa mulai sadar: hidup di dunia IT itu… tidak pernah benar-benar selesai.