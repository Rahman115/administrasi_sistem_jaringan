# administrasi_sistem_jaringan

## Intoduction

## Permasalahan dalam penginstalan Debian

``` sudo apt update ```

### Daftarkan CD-ROM

jalankan perintah scan ```sudo apt-cdrom add```

### File konfigurasi repositori

perintah ```sudo nano /etc/apt/sources.list```

```
> Error : failed to mount 'dev/sr0' to /media/cdrom/

sudo mkdir -p /media/cdrom

> Paksa Mount secara Manual

sudo mount /dev/sr0 /media/cdrom

sudo apt-cdrom add

sudo apt update
```

https://chatgpt.com/share/6ab3479c-e638-83ec-8c51-9a3dbf7adfa4
