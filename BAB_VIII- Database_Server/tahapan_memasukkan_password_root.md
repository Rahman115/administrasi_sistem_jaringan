# Tahapan dalam Memasukan Password Root


Setelah kamu menjalankan perintah sudo mariadb-secure-installation, kamu akan dipandu melalui beberapa pertanyaan keamanan. Berikut adalah penjelasan tiap tahapannya agar kamu tidak bingung memilih opsi:
## 1. Masukkan Password Root Saat Ini

* Prompt: Enter current password for root (enter for none):
* Penjelasan: Jika kamu baru saja menginstal MariaDB dan belum pernah mengatur password, cukup tekan Enter.

## 2. Beralih ke Otentikasi unix_socket (Opsional)

* Prompt: Switch to unix_socket authentication [Y/n]
* Saran: Pilih n jika kamu ingin menggunakan password biasa untuk login. Pilih Y jika kamu ingin login ke MariaDB secara otomatis selama kamu menggunakan user sistem (sudo).

## 3. Mengubah Password Root

* Prompt: Change the root password? [Y/n]
* Saran: Pilih Y.
* Tindakan: Masukkan password baru yang kuat dan ulangi untuk konfirmasi. Ini adalah password utama untuk akses administratif database.

## 4. Menghapus Pengguna Anonim

* Prompt: Remove anonymous users? [Y/n]
* Saran: Pilih Y.
* Penjelasan: Ini menghapus user "kosong" yang memungkinkan siapa saja masuk ke database tanpa akun, sehingga meningkatkan keamanan.

## 5. Membatasi Login Root Jarak Jauh

* Prompt: Disallow root login remotely? [Y/n]
* Saran: Pilih Y.
* Penjelasan: Ini memastikan user root hanya bisa login dari server lokal (localhost). Untuk akses jarak jauh, sebaiknya gunakan user biasa dengan hak akses terbatas.

## 6. Menghapus Database 'test'

* Prompt: Remove test database and access to it? [Y/n]
* Saran: Pilih Y.
* Penjelasan: MariaDB secara default membuat database bernama test yang bisa diakses siapa saja. Menghapusnya adalah standar keamanan server produksi.

## 7. Memuat Ulang Tabel Hak Akses (Privileges)

* Prompt: Reload privilege tables now? [Y/n]
* Saran: Pilih Y.
* Penjelasan: Ini akan langsung menerapkan semua perubahan yang baru saja kamu buat tanpa perlu merestart layanan MariaDB.


