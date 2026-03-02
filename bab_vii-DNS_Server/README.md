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
# 📌 PROSEDUR STANDAR INSTALASI DNS SERVER (BIND9)

---

## 1️⃣ Perencanaan Awal (Ini yang sering dilewati, lalu menyesal)

Sebelum install, tentukan:

* IP server (misal: `192.168.10.2`)
* Domain yang akan dibuat (misal: `lab.local`)
* Apakah:

  * Authoritative saja?
  * Sekaligus caching?
  * Ada reverse DNS?

Tanpa perencanaan, konfigurasi hanya jadi kumpulan file tanpa arah.

---

## 2️⃣ Set IP Static

DNS server wajib punya IP tetap.

Edit:

```bash
/etc/network/interfaces
```

Contoh:

```bash
auto ens33
iface ens33 inet static
    address 192.168.10.2
    netmask 255.255.255.0
    gateway 192.168.10.1
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

---

## 3️⃣ Instalasi Paket BIND9

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

## 4️⃣ Struktur File Penting

File utama konfigurasi ada di:

```
/etc/bind/
```

File penting:

* `named.conf`
* `named.conf.options`
* `named.conf.local`
* `db.local`
* `db.127`

Biasanya zona baru ditambahkan di `named.conf.local`.

---

## 5️⃣ Konfigurasi Forward Zone

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

## 6️⃣ Buat File Database Zona

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

## 7️⃣ Konfigurasi Reverse Zone (Standar Profesional)

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

## 8️⃣ Validasi Konfigurasi (Bagian yang Menyelamatkan Harga Diri)

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

## 9️⃣ Restart Service

```bash
systemctl restart bind9
```

---

## 🔟 Testing

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
