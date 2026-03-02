# DNS Server


---

## 📖 Alur Besar Instalasi DNS Server

1. Perencanaan domain & IP
2. Set IP static
3. Install BIND9
4. Konfigurasi forward zone
5. Konfigurasi reverse zone
6. Validasi
7. Restart
8. Testing

Urutannya tidak boleh dibalik.

---
## 📌 PROSEDUR STANDAR INSTALASI DNS SERVER (BIND9)
### 1️⃣ Perencanaan Awal

Sebelum install, tentukan:

* IP server (misal: `192.168.10.2`)
* Domain yang akan dibuat (misal: `lab.local`)
* Apakah:

  * Authoritative saja?
  * Sekaligus caching?
  * Ada reverse DNS?

Tanpa perencanaan, konfigurasi hanya jadi kumpulan file tanpa arah.

---

### 2️⃣ Set IP Static

DNS server wajib punya IP tetap.

1. Metode 1 : Langsung di File Konfigurasi Antarmuka
Ini adalah metode yang disarankan karena pengaturannya akan persisten meskipun jaringan direstart.
Edit:

```bash
nano /etc/network/interfaces
```
Pada bagian konfigurasi antarmuka Anda (eth0 atau ens33), tambahkan baris `dns-nameservers` diikuti dengan alamat DNS yang ingin Anda gunakan . Anda dapat menambahkan lebih dari satu dengan spasi.
Contoh:

```text
auto ens33
iface ens33 inet static
    address 192.168.10.2
    netmask 255.255.255.0
    gateway 192.168.10.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

Restart network:

```bash
systemctl restart networking
```

Cek:

```bash
ip a
```

Pastikan IP tidak berubah-ubah seperti janji mantan.
2. Metode 2: Mengedit File `/etc/resolv.conf`
Metode ini lebih langsung, namun pengaturannya bisa ditimpa oleh sistem lain.

Buka file `/etc/resolv.conf`:

```bash
sudo nano /etc/resolv.conf
```

Tambahkan baris `nameserver` diikuti alamat DNS yang diinginkan . Setiap `nameserver` ditulis di baris baru.

```text
nameserver 8.8.8.8
nameserver 8.8.4.4
```

Simpan file. Perubahan akan langsung berlaku.

---

### 3️⃣ Instalasi Paket BIND9

Update repository:

```bash
apt update
```

Install:

```bash
apt install bind9 bind9utils bind9-doc -y
```

Cek status:

```bash
systemctl status bind9
```

Jika aktif, server sudah hidup. Tapi belum tahu harus menjawab apa.

---

### 4️⃣ Struktur File Penting

File utama konfigurasi ada di:

```
/etc/bind/
```

File penting, untuk melihat lakukan `ls` :

* `named.conf`
* `named.conf.options`
* `named.conf.local`
* `db.local`
* `db.127`

Biasanya zona baru ditambahkan di `named.conf.local`.

---

### 5️⃣ Konfigurasi Forward Zone

Edit:

```bash
nano /etc/bind/named.conf.local
```

Tambahkan:

```
zone "lab.local" {
    type master;
    file "/etc/bind/db.lab";
};
```

---

### 6️⃣ Buat File Database Zona

Copy template:

```bash
cp /etc/bind/db.local /etc/bind/db.lab
```

Edit:

```bash
nano /etc/bind/db.lab
```

Contoh isi:

```
$TTL    604800
@       IN      SOA     ns.lab.local. admin.lab.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      ns.lab.local.
ns      IN      A       192.168.10.2
www     IN      A       192.168.10.2
```

Penjelasan:

* SOA = Start of Authority
* NS = Name Server
* A = mapping domain ke IP

Serial wajib naik setiap ada perubahan.

---

### 7️⃣ Konfigurasi Reverse Zone (Standar Profesional)

Edit lagi:

```bash
nano /etc/bind/named.conf.local
```

Tambahkan:

```
zone "10.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.reverse";
};
```

Copy template:

```bash
cp /etc/bind/db.127 /etc/bind/db.reverse
```

Edit:

```
$TTL    604800
@       IN      SOA     ns.lab.local. admin.lab.local. (
                              2
                         604800
                          86400
                        2419200
                         604800 )

@       IN      NS      ns.lab.local.
2       IN      PTR     ns.lab.local.
```

Angka `2` berasal dari IP `192.168.10.2`.

---

### 8️⃣ Validasi Konfigurasi (Bagian yang Menyelamatkan Harga Diri)

Cek syntax:

```bash
named-checkconf
```

Cek zona:

```bash
named-checkzone lab.local /etc/bind/db.lab
```

Kalau error, jangan restart dulu. Perbaiki.

---

### 9️⃣ Restart Service

```bash
systemctl restart bind9
```

---

### 🔟 Testing

Install dig:


```bash
apt install dnsutils
```

Tes forward:

```bash
dig @192.168.10.2 www.lab.local
```


Tes reverse:

```bash
dig -x 192.168.10.2
```

Jika jawabannya benar, DNS bekerja.

Jika tidak, kembali ke langkah 8.
DNS itu disiplin. Ia tidak mentolerir typo.

---


## 📌 Alur Besar DDNS

1. Install BIND
2. Install DHCP
3. Buat TSIG key
4. Integrasikan key ke DNS
5. Integrasikan key ke DHCP
6. Pindahkan file zona ke direktori writable
7. Restart service
8. Test dengan client

---

### 📌 KONSEP DASAR DDNS

Dynamic DNS memungkinkan:

* DHCP memberikan IP
* DHCP otomatis mengupdate DNS
* Record A dan PTR dibuat tanpa kita edit file zona manual

Artinya:
Client hidup → dapat IP → langsung punya nama di DNS.

Lebih elegan. Lebih hidup.

---

### 📌 SKENARIO

Server IP: `192.168.10.2`
Domain: `lab.local`
DHCP melayani: `192.168.10.100-200`

---

### 1️⃣ Install DHCP Server

```bash
apt install isc-dhcp-server -y
```

Edit interface:

```bash
nano /etc/default/isc-dhcp-server
```

Isi:

```
INTERFACESv4="ens33"
```

---

### 2️⃣ Buat TSIG Key (Kunci Rahasia DNS)

DNS tidak akan menerima update sembarangan.
Kita perlu kunci autentikasi.

Buat key:

```bash
tsig-keygen -a HMAC-SHA256 ddns-key
```

Output contoh:

```
key "ddns-key" {
    algorithm hmac-sha256;
    secret "XXXXXXXXXXXXXXXXXXXXXXXX";
};
```

Simpan isi itu di:

```bash
nano /etc/bind/ddns.key
```

Amankan:

```bash
chmod 640 /etc/bind/ddns.key
chown root:bind /etc/bind/ddns.key
```

---

### 3️⃣ Hubungkan Key ke BIND

Edit:

```bash
nano /etc/bind/named.conf.local
```

Tambahkan:

```
include "/etc/bind/ddns.key";
```

Lalu ubah zona forward:

```
zone "lab.local" {
    type master;
    file "/var/lib/bind/db.lab";
    allow-update { key ddns-key; };
};
```

Perhatikan:
File zona pindah ke `/var/lib/bind/` supaya bisa ditulis oleh BIND.

Lakukan hal sama untuk reverse zone.

---

### 4️⃣ Ubah Permission Folder Zona

```bash
chown bind:bind /var/lib/bind/
chmod 775 /var/lib/bind/
```

DNS perlu hak tulis. Jangan pelit izin.

---

### 5️⃣ Konfigurasi DHCP agar Update DNS

Edit:

```bash
nano /etc/dhcp/dhcpd.conf
```

Tambahkan di bagian atas:

```
ddns-update-style interim;
ignore client-updates;

key ddns-key {
    algorithm hmac-sha256;
    secret "XXXXXXXXXXXXXXXXXXXXXXXX";
}

zone lab.local. {
    primary 127.0.0.1;
    key ddns-key;
}

zone 10.168.192.in-addr.arpa. {
    primary 127.0.0.1;
    key ddns-key;
}
```

Lalu tambahkan subnet:

```
subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.100 192.168.10.200;
    option domain-name "lab.local";
    option domain-name-servers 192.168.10.2;
    option routers 192.168.10.1;
}
```

---

### 6️⃣ Restart Service

Urutannya penting:

```
systemctl restart bind9
systemctl restart isc-dhcp-server
```

Kalau dibalik dan gagal, jangan salahkan semesta.

---

### 7️⃣ Testing

Hubungkan client ke DHCP.

Di client:

```bash
ipconfig /release
ipconfig /renew
```

Lalu di server:

```bash
dig client01.lab.local
```

Atau cek file zona di:

```bash
/var/lib/bind/db.lab
```

Kalau berhasil, record akan muncul otomatis.
