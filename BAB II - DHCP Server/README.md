# BAB II DHCP Server

Tentu. Berikut tutorial praktik DHCP Server Debian 10 di VirtualBox dari awal sampai pengujian client. Saya buat dengan skenario yang sederhana agar mudah dipraktikkan.

1. Topologi Praktik

Kita akan menggunakan 2 VM:

VirtualBox
                 Internal Network
                 "dhcp-lab"
                       |
             +---------+---------+
             |                   |
        DHCP SERVER          DHCP CLIENT
         Debian 10             Debian 10
       192.168.10.1               |
             |                     |
             +---------------------+
                    DHCP

Konfigurasi yang digunakan

Perangkat	IP	Keterangan

Server	192.168.10.1	IP static
Client	DHCP	Mendapat IP otomatis
Network	192.168.10.0/24	Jaringan praktik
DHCP range	192.168.10.100 - 192.168.10.200	IP untuk client
Gateway	192.168.10.1	Gateway
DNS	8.8.8.8	DNS



---

A. Membuat VM Server

Buat satu VM dengan nama misalnya:

DHCP-SERVER

Gunakan Debian 10 sebagai sistem operasi.

Untuk praktik, RAM 1 GB dan storage sekitar 10–20 GB sudah cukup.

2. Konfigurasi Network Server di VirtualBox

Matikan VM terlebih dahulu.

Masuk:

Settings → Network

Pada Adapter 1:

Enable Network Adapter     ✓
Attached to:               Internal Network
Name:                      dhcp-lab

Jadi:

Adapter 1
    |
    +-- Internal Network
            |
            +-- dhcp-lab

Kita menggunakan Internal Network supaya server dan client dapat berkomunikasi dalam jaringan virtual yang sama tanpa bergantung pada jaringan internet.


---

B. Membuat VM Client

Buat VM kedua:

DHCP-CLIENT

Gunakan Debian 10 juga.

Pada konfigurasi VirtualBox:

Settings → Network → Adapter 1

Pilih:

Enable Network Adapter     ✓
Attached to:               Internal Network
Name:                      dhcp-lab

Penting: nama Internal Network pada Server dan Client harus sama:

dhcp-lab

Sehingga:

SERVER
   |
   | Internal Network: dhcp-lab
   |
CLIENT


---

C. Konfigurasi Server Debian 10

Login ke Debian 10 Server.

3. Masuk sebagai root

su -

Kemudian masukkan password root.

Atau jika menggunakan sudo:

sudo -i


---

4. Cek nama interface jaringan

Jalankan:

ip addr

Contohnya:

1: lo: <LOOPBACK>
    inet 127.0.0.1/8

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.10.1/24

Pada tutorial ini kita menggunakan:

enp0s3

Catatan: nama interface pada VM Anda mungkin berbeda. Bisa saja ens33, enp0s3, eth0, dan sebagainya.

Gunakan nama interface milik Anda pada langkah berikutnya.


---

D. Memberikan IP Static ke Server

5. Edit konfigurasi network

nano /etc/network/interfaces

Tambahkan/ubah menjadi:

auto lo
iface lo inet loopback

auto enp0s3
iface enp0s3 inet static
    address 192.168.10.1
    netmask 255.255.255.0

Jika interface Anda bukan enp0s3, sesuaikan.

Contoh jika interface Anda ens33:

auto ens33
iface ens33 inet static
    address 192.168.10.1
    netmask 255.255.255.0

Simpan:

Ctrl + O
Enter
Ctrl + X

Restart networking:

systemctl restart networking

Kemudian cek:

ip addr

Pastikan terdapat:

inet 192.168.10.1/24


---

E. Instalasi DHCP Server

6. Update repository

apt update

Kemudian instal:

apt install isc-dhcp-server -y

Paket yang digunakan adalah:

isc-dhcp-server


---

F. Menentukan Interface DHCP

7. Edit konfigurasi interface DHCP

Buka:

nano /etc/default/isc-dhcp-server

Cari:

INTERFACESv4=""

Ubah menjadi:

INTERFACESv4="enp0s3"

Jika interface Anda ens33, gunakan:

INTERFACESv4="ens33"

Simpan dan keluar.


---

G. Konfigurasi DHCP Server

8. Backup konfigurasi

Sebaiknya buat backup terlebih dahulu:

cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.backup

Kemudian edit:

nano /etc/dhcp/dhcpd.conf

Masukkan konfigurasi berikut:

option domain-name "dhcp-lab.local";
option domain-name-servers 8.8.8.8, 8.8.4.4;

default-lease-time 600;
max-lease-time 7200;

authoritative;

subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.100 192.168.10.200;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.10.255;
    option routers 192.168.10.1;
}

Simpan:

Ctrl + O
Enter
Ctrl + X


---

H. Penjelasan Konfigurasi DHCP

Bagian:

subnet 192.168.10.0 netmask 255.255.255.0

menentukan network yang digunakan.

Kemudian:

range 192.168.10.100 192.168.10.200;

berarti DHCP dapat memberikan IP:

192.168.10.100
192.168.10.101
192.168.10.102
...
192.168.10.200

Sedangkan:

option routers 192.168.10.1;

memberikan gateway kepada client.

Dan:

option domain-name-servers 8.8.8.8, 8.8.4.4;

memberikan DNS kepada client.


---

I. Mengecek Konfigurasi DHCP

Sebelum menjalankan DHCP, lakukan pengecekan:

dhcpd -t -cf /etc/dhcp/dhcpd.conf

Jika konfigurasi benar, biasanya tidak muncul error.

Jika muncul error seperti:

/etc/dhcp/dhcpd.conf line xx: ...

berarti ada kesalahan pada konfigurasi.

Periksa kembali:

nano /etc/dhcp/dhcpd.conf


---

J. Menjalankan DHCP Server

9. Restart service

systemctl restart isc-dhcp-server

Kemudian:

systemctl status isc-dhcp-server

Jika berhasil, akan terlihat:

Active: active (running)

Contoh:

● isc-dhcp-server.service
   Loaded: loaded
   Active: active (running)

Tekan:

q

untuk keluar dari tampilan status.


---

K. Mengaktifkan DHCP Saat Boot

Jalankan:

systemctl enable isc-dhcp-server

Cek:

systemctl is-enabled isc-dhcp-server

Hasil yang diharapkan:

enabled


---

L. Konfigurasi Client Debian 10

Sekarang pindah ke VM:

DHCP-CLIENT

10. Cek interface

ip addr

Misalnya:

2: enp0s3:


---

11. Atur client agar menggunakan DHCP

Edit:

nano /etc/network/interfaces

Gunakan:

auto lo
iface lo inet loopback

auto enp0s3
iface enp0s3 inet dhcp

Sesuaikan enp0s3 dengan interface Anda.

Simpan kemudian restart networking:

systemctl restart networking


---

M. Meminta IP dari DHCP Server

Pada client jalankan:

dhclient -v enp0s3

Perhatikan outputnya.

Jika berhasil, akan ada proses seperti:

DHCPDISCOVER
DHCPOFFER
DHCPREQUEST
DHCPACK

Ini merupakan proses penting dalam DHCP.

Secara sederhana:

CLIENT                  SERVER

DHCPDISCOVER  -------->

              <-------- DHCPOFFER

DHCPREQUEST   -------->

              <-------- DHCPACK


---

N. Mengecek IP Client

Jalankan:

ip addr

Seharusnya client mendapatkan IP seperti:

192.168.10.100/24

atau:

192.168.10.101/24

atau IP lain dalam rentang:

192.168.10.100 - 192.168.10.200


---

O. Mengecek Gateway

Jalankan pada client:

ip route

Hasil yang diharapkan kurang lebih:

default via 192.168.10.1 dev enp0s3
192.168.10.0/24 dev enp0s3

Artinya DHCP server berhasil memberikan gateway:

192.168.10.1


---

P. Pengujian Koneksi Client → Server

Dari client, lakukan:

ping 192.168.10.1

Jika berhasil:

64 bytes from 192.168.10.1:
64 bytes from 192.168.10.1:
64 bytes from 192.168.10.1:

Berarti:

Client
  |
  | ping
  v
192.168.10.1
  |
Server

sudah terhubung.


---

Q. Melihat Lease DHCP

Kembali ke DHCP Server.

Jalankan:

cat /var/lib/dhcp/dhcpd.leases

Anda dapat melihat client yang mendapatkan IP.

Contohnya:

lease 192.168.10.100 {
  starts 3 2026/09/23 02:00:00;
  ends 3 2026/09/23 02:10:00;
  tstp 3 2026/09/23 02:10:00;
  binding state active;
}

Ini menunjukkan DHCP server telah memberikan:

192.168.10.100

kepada sebuah client.


---

R. Melihat Log DHCP

Jika ingin melihat aktivitas DHCP:

journalctl -u isc-dhcp-server

Atau:

journalctl -u isc-dhcp-server -f

Perintah -f akan menampilkan log secara realtime.

Kemudian pada client jalankan:

dhclient -v enp0s3

Anda dapat melihat aktivitas DHCP secara langsung pada server.


---

S. Pengujian Mendapatkan IP Baru

Untuk menguji kembali proses DHCP pada client:

dhclient -r enp0s3

Perintah tersebut melepaskan IP DHCP.

Kemudian:

dhclient -v enp0s3

Client akan kembali meminta IP kepada server.

Cek:

ip addr


---

T. Pengujian DNS

Karena konfigurasi DHCP kita memberikan:

8.8.8.8
8.8.4.4

Anda dapat mengeceknya dengan:

cat /etc/resolv.conf

Namun karena kita menggunakan Internal Network, client belum tentu dapat mengakses internet. Itu tidak berarti DHCP gagal.

Untuk praktik DHCP dasar, yang penting adalah client memperoleh:

IP address

subnet mask

gateway

DNS


dari DHCP server.


---

U. Troubleshooting

1. DHCP Server failed

Cek:

systemctl status isc-dhcp-server

Kemudian:

journalctl -u isc-dhcp-server

Cek konfigurasi:

dhcpd -t -cf /etc/dhcp/dhcpd.conf


---

2. Client tidak mendapatkan IP

Pertama periksa VirtualBox.

Server:

Internal Network: dhcp-lab

Client:

Internal Network: dhcp-lab

Keduanya harus sama.

Kemudian pada server:

ip addr

Pastikan server mempunyai:

192.168.10.1/24

Cek interface DHCP:

cat /etc/default/isc-dhcp-server

Pastikan:

INTERFACESv4="enp0s3"

sesuai dengan interface sebenarnya.


---

3. IP client 169.254.x.x

Jika client mendapatkan:

169.254.x.x

biasanya client tidak mendapatkan respons DHCP.

Periksa:

VirtualBox Network
        ↓
Internal Network
        ↓
Interface Server
        ↓
IP Server
        ↓
isc-dhcp-server
        ↓
dhcpd.conf


---

V. Checklist Praktikum

Gunakan checklist berikut saat praktik:

Server

[✓] Debian 10 terinstall
[✓] VirtualBox Internal Network
[✓] Nama network = dhcp-lab
[✓] IP server = 192.168.10.1
[✓] isc-dhcp-server terinstall
[✓] INTERFACESv4 sudah benar
[✓] dhcpd.conf sudah dikonfigurasi
[✓] dhcpd -t tidak error
[✓] isc-dhcp-server active (running)

Client

[✓] Debian 10 terinstall
[✓] Internal Network = dhcp-lab
[✓] Interface menggunakan DHCP
[✓] Mendapat IP 192.168.10.x
[✓] Mendapat gateway 192.168.10.1
[✓] Bisa ping 192.168.10.1


---

Hasil Akhir

Jika semuanya berhasil, konfigurasi akhirnya akan seperti ini:

VIRTUALBOX
                  |
           Internal Network
              dhcp-lab
                  |
        +---------+---------+
        |                   |
        |                   |
   DHCP SERVER          DHCP CLIENT
   Debian 10            Debian 10
        |                   |
   192.168.10.1        192.168.10.100
        |                   |
        +-------------------+
             DHCP

Server:

IP       : 192.168.10.1
Netmask  : 255.255.255.0
DHCP     : 192.168.10.100-200

Client:

IP       : 192.168.10.100
Netmask  : 255.255.255.0
Gateway  : 192.168.10.1
DNS      : 8.8.8.8

Urutan paling penting untuk diingat:

VirtualBox
    ↓
Internal Network
    ↓
IP Static Server
    ↓
Install isc-dhcp-server
    ↓
Tentukan interface
    ↓
Konfigurasi dhcpd.conf
    ↓
Test konfigurasi
    ↓
Restart DHCP
    ↓
Client = DHCP
    ↓
dhclient
    ↓
Cek IP
    ↓
Ping Server

Kalau ini untuk laporan praktikum, struktur di atas juga bisa dijadikan BAB Praktikum: Tujuan, Alat dan Bahan, Topologi, Langkah Kerja, Pengujian, Hasil, Analisis, dan Kesimpulan.