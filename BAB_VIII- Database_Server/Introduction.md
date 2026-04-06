# Instalasi MariaDB Server

##Instalasi MariaDB di Debian (versi 11 atau 12) dapat dilakukan dengan mudah melalui repositori resmi Debian atau repositori MariaDB untuk versi terbaru. Secara default, Debian 12 menyertakan MariaDB 10.11, sementara Debian 11 menyertakan versi 10.5. [1, 2, 3, 4] 
## Langkah-Langkah Instalasi Standar

   1. Perbarui Sistem
   Pastikan daftar paket di sistem Anda adalah yang terbaru sebelum memulai instalasi.
   ```path

   sudo apt update && sudo apt upgrade -y

```
   
   2. Instal Paket MariaDB
   Gunakan perintah berikut untuk menginstal server dan klien MariaDB.
   ```path

   sudo apt install mariadb-server mariadb-client -y

   ```
   3. Verifikasi Status Layanan
   Setelah instalasi selesai, periksa apakah layanan MariaDB sudah berjalan.
   ```path

   sudo systemctl status mariadb

   ```
   [1, 3, 5, 6] 

## Konfigurasi Keamanan (Penting)
Sangat disarankan untuk menjalankan skrip keamanan bawaan guna mengamankan instalasi Anda, seperti mengatur kata sandi root, menghapus pengguna anonim, dan mematikan login root jarak jauh. [7, 8, 9] 
```path

sudo mysql_secure_installation

# jika hasilnya commad not found,
# maka jalankan perintah

sudo mariadb-secure-instalation

```
Ikuti petunjuk di layar (tekan Y untuk sebagian besar opsi keamanan yang disarankan). [5] 
## Akses MariaDB
Untuk masuk ke konsol MariaDB sebagai pengguna root, gunakan perintah berikut: [6, 10, 11] 

sudo mariadb -u root -p

## Tips Tambahan

* Versi Terbaru: Jika Anda memerlukan versi yang lebih baru (seperti MariaDB 11), Anda perlu menambahkan repositori resmi dari MariaDB.org terlebih dahulu sebelum melakukan instalasi.
* Manajemen Layanan: Gunakan sudo systemctl start mariadb, stop, atau restart untuk mengelola status database Anda sesuai kebutuhan. [6, 12, 13] 


[1] [digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-debian-11)
[2] [https://www.youtube.com](https://www.youtube.com/watch?v=tcT_40S_Uig)
[3] [https://linuxcapable.com](https://linuxcapable.com/how-to-install-mariadb-debian-linux/)
[4] [https://mariadb.com](https://translate.google.com/translate?u=https://mariadb.com/docs/general-resources/distributions-including-mariadb&hl=id&sl=en&tl=id&client=sge#:~:text=Distribusi%20Linux%20MariaDB%2010.6%20telah%20tersedia%20sejak,Debian%2012%20%22Bookworm%22%20MariaDB%2010.11%20sebagai%20default.)
[5] [https://www.youtube.com](https://www.youtube.com/watch?v=39ilTkhJstk)
[6] [https://www.youtube.com](https://www.youtube.com/watch?v=GF_seqfoZHk)
[7] [https://www.youtube.com](https://www.youtube.com/watch?v=qpVz4r_ICOs)
[8] [https://www.youtube.com](https://www.youtube.com/watch?v=1LN1JSoO2ew)
[9] [https://www.ionos.com](https://www.ionos.com/digitalguide/hosting/technical-matters/install-mariadb-on-debian-12/)
[10] [https://www.youtube.com](https://www.youtube.com/watch?v=39ilTkhJstk)
[11] [https://id.scribd.com](https://id.scribd.com/document/515284810/3-MODUL-2-DDL)
[12] [https://www.tecmint.com](https://www.tecmint.com/install-mariadb-in-debian/)
[13] [https://www.ionos.ca](https://www.ionos.ca/digitalguide/hosting/technical-matters/install-mariadb-on-debian-11/)
