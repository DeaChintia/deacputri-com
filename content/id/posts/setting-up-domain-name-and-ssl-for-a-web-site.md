---
title: 'Mengkonfigurasi Nama Domain dan SSL pada Situs Web dengan Namecheap dan Cerbot'
date: '2024-11-20T20:57:00+07:00'
categories: ['Web Hosting']
tags: ['Hugo', 'Web Hosting', 'SSL']
series: ['Situs Web Personal Pakai Hugo']
draft: false
---

Di post sebelumnya, saya menulis tentang cara membuat situs web memakai Hugo. Sekarang, bagaimana caranya kalau saya ingin orang lain bisa akses situs saya? Setelah meng-*hosting* situs web di VPS, sebenarnya situs web sudah dapat diakses dari mana saja menggunakan alamat IP http://144.21.57.200 dari browser. Tetapi, mengingat alamat situs web dengan alamat IP sangat sulit. Bayangkan kalau saya ingin teman saya mengakses situs web ini, lalu mereka nanya "Okee, *link*-nya apa?" dan saya jawab, "Oh, *link*-nya H-T-T-P titik dua *double slash* satu-empat-empat titik dua-satu titik lima-tujuh titik dua ratus." Mereka pasti bakal jawab, "Hah?" Oleh karena itu, sebuah situs web membutuhkan yang namanya nama domain.

## Nama Domain

### Apa itu Nama Domain?
Nama domain itu seperti alias dari alamat IP sebuah situs web. Ada komputer-komputer di dunia yang akan mengingat nama domain dari setiap alamat IP yang sudah terdaftar. Komputer-komputer ini disebut sebagai *Domain Name Server* (DNS). Situs web saya memiliki nama domain deacputri.com dan nama domain itu mengarah ke alamat IP "144.21.57.200". Untuk memiliki sebuah nama domain, kita harus "membelinya" dari *domain name registrar*. Saat ini saya menggunakan Namecheap untuk membeli nama domain dikarenakan harga yang rendah dan jasa "*whois protection*" gratis.

Apa itu "*whois protection*"? Saat saya registrasi untuk "membeli" nama domain, saya diwajibkan untuk mengisi sebuah form dengan data-data personal saya (seperti nama, nomor telepon, alamat, dll). Data ini akan disimpan secara publik pada basis data (*database*) WHOIS dan semua orang bisa melihat data-data ini untuk mengecek apakah sebauh nama domain sudah dipakai atau belum. Tentunya menaruh data personal secara publik itu ide yang sangat buruk (bisa kena spam dan sebagainya). Dengan "*whois protection*", *registrar* akan mengisi form tersebut dengan data dummy. Saran saya, selalu pilih *registrar* yang menyediakan "*whois protection*" gratis seperti Cloudflare dan Namecheap.

### Mengkonfigurasi Nama Domain
Pertama, daftar ke sebuah *registrar* dan beli sebuah nama domain. Setelah saya memiliki nama domain, saya bisa melakukan konfigurasi nama domain pada "*Host Records*" dengan detail berikut:

| Type         | Host                      | Value                 |
| ------------ | ------------------------- | --------------------- |
| A Record     | nama-domain-dasar.com     | [alamat IP server]    |
| CNAME Record | www.nama-domain-dasar.com | nama-domain-dasar.com |

Berikut contohnya pada *Namecheap Advanced DNS*:

![*Namecheap Advanced DNS*](/images/setting-up-domain-name-and-ssl-for-a-web-site/namecheap-advanced-dns.png)

Catatan:
- @ merujuk pada nama domain dasar (`deacputri.com`)
- www merujuk pada `www.deacputri.com`

Simpan konfigurasi dan tunggu sampai nama domain diperbarui pada berbagai server DNS di dunia (dapat dicek pada dnschecker.org)

Berikut tampilan *DNS Check* untuk A *record* dan CNAME *record*.


![Cek DNS untuk A *record*](/images/setting-up-domain-name-and-ssl-for-a-web-site/dns-check-for-a-record.png)

![Cek DNS untuk CNAME *record*](/images/setting-up-domain-name-and-ssl-for-a-web-site/dns-check-for-cname-record.png)

Ada beberapa catatan untuk A *record* dan CNAME *record*:
- A *record* (*Address Record*) adalah catatan (*record*) yang mengasosiasikan nama domain atau subdomain dengan sebuah alamat IP. Pada contoh di atas, `deacputri.com` diasosiasikan dengan alamat IP = `144.21.57.200`
- CNAME *record* (*Canonical Name Record*) adalah alias dari sebuah nama domain. Pada contoh di atas, `www.deacputri.com` adalah alias dari `deacputri.com`. CNAME *record* tidak boleh diasosiasikan dengan nama domain dasar karena spesifikasi DNS tidak mengizinkan CNAME *record* ada bersamaan dengan *record* lain yang merujuk ke nama domain yang sama. Misalnya, jika saya mengkonfigurasi CNAME *record* untuk `example.com`, saya tidak bisa punya konfigurasi record berikut:
  - `example.com.  A      192.0.2.1`
  - `example.com.  MX     mail.example.com.`

## HTTPS dan SSL

### Apa itu HTTPS dan SSL?
HTTP adalah versi amannya HTTP (*Hyper Text Transfer Protocol*), protokol aplikasi yang digunakan untuk memuat halaman situs web. HTTPS diamankan menggunakan protokol yang disebut sebagai SSL (disebut juga TLS, *Transport Layer Security*). Protokol SSL mengenkripsi data yang dikirim antara browser dan situs web sehingga tidak ada yang tahu apa data tersebut saat data tersebut diintip. Setifikat SSL dibutuhkan untuk membangun koneksi SSL antara browser dan server web. Berikut cara SSL bekerja:

1. **Browser mencoba untuk konek ke situs web menggunakan HTTPS (diamankan menggunakan SSL/TLS).**
   Menggunakan https sebagai link artinya saya menginstruksikan browser kalau koneksi yang ada harus aman.
2. **Server web mengirimkan sertifikat SSL ke browser.**
   Sertifikat berisi dua hal penting berikut:
   - Identitas situs web (contoh: nama domain).
   - Informasi tentang *Certificate Authorithy* (CA), sebuah lembaga terpercaya yang mengeluarkan sertifikat tersebut dan menjamin bahwa situs web tersebut asli.
3. **Browser mengecek sertifikat.**
   Browser akan memverifikasi:
   - bahwa sertifikatnya valid (tidak kadaluarsa atau palsu).
   - bahwa *Certificate Authority*-nya (CA) terpercaya dan diketahui.
4. **Jika sertifikatnya valid, browser akan membuat *key* sementara.**
   - *Key* ini disebut juga sebagai *session key*, dan merupakan kunci satu kali pakai yang akan digunakan untuk mengenkripsi semua data pada sesi tersebut.
   - Browser akan mengenkripsi *session key* menggunakan *public key* milik server situs web, yang sudah tersedia pada sertifikat SSL.

   Catatan penting:
   - *Public key* hanya akan digunakan untuk mengirimkan *session key* ke server melalui koneksi yang aman.
   - *Public key* adalah kunci yang dapat digunakan oleh semua orang untuk mengenkripsi informasi. Kunci ini dibagikan kepada semua orang dan tercatat pada sertifikat SSL.
   - Setelah itu, *session key* akan digunakan untuk mengenkripsi dan mendekripsi data yang saling ditukar pada saat sesi berlangsung.
5. **Server akan mendekripsi *session key* menggunakan *private key* yang dimiliki oleh server.**
   - *Private key* adalah kunci rahasia yang hanya diketahui oleh server. Kunci ini digunakan untuk mendekripsi informasi yang dienkripsi menggunakan *public key*.
   - Jadi pada langkah ini, server menggunakan *private key* miliknya untuk mendekripsi *session key* yang dikirim oleh browser.
6. **Setelah *session key* berhasil dibagikan ke server, kunci tersebut digunakan untuk mengenkripsi dan mendekripsi data antara browser dan server.**
   Setelah langkah ini, browser dan server akan memiliki *session key*. Kunci ini akan memungkinkan browser dan server untuk:
   - Mengenkripsi data sebelum mengirimkannya, sehingga data aman dari pencuri data.
   - Mendekripsi data yang diterima, sehingga server bisa membaca informasi yang dikirimkan.

### Mengkonfigurasi HTTPS dan Sertifikat SSL
Untuk mengkonfigurasi HTTP dan sertifikat SSL, saya menggunakan Certbot. Certbot memilik instruksi pada web mereka untuk berbagai macam software (web server saya menggunakan Apache) dan beberapa sistem operasi. Pada contoh yang saya tulis, saya akan menggunakan instruksi untuk Linux dengan snap untuk menginstal Certbot. Berikut langkah-langkah untuk menginstal sertifikat SSL dengan Certbot pada Apache di dalam sistem operasi Ubuntu:
1. Akses server menggunakan SSH memakai *user* dengan hak akses sudo.
2. Instal snapd (Ubuntu sudah punya dari awal).
   ```bash
   sudo apt update
   sudo apt install snapd
   ```
3. *Logout* dan *login* kembali.
4. Instal Certbot.
   ```bash
   sudo snap install --classic certbot
   ```
5. Siapkan perintah `certbot` untuk memastikan perintah `certbot` dapat dijalankan.
   ```bash
   sudo ln -s /snap/bin/certbot /usr/bin/certbot
   ```
6. Dapatkan dan install sertifikat.
   ```bash
   sudo certbot --apache
   ```
7. Tes perbaruan otomatis.
   Certbot akan menginstal *scheduler* dengan cron job atau systemmd timer yang akan memperbarui sertifikat secara otomatis sebelum sertifikat tersebut kadaluarsa. Sehingga saya tidak perlu lagi menjalankan `certbot` untuk memperbarui sertifikat. Untuk mengetes pembaruan otomatis sertifikat, jalankan perintah berikut:
    ```bash
   sudo certbot renew --dry-run
   ```
   Perintah di atas terinstal pada salah satu dari lokasi-lokasi *scheduler* berikut:
   - `/etc/crontab/`
   - `/etc/cron.*/*`
   - `systemctl list-timers`
8. Konfirmasi bahwa sertifikat SSL sudah terinstal secara benar dengan mengakses situs web menggunakan protokol HTTPS (contoh: https://deacputri.com).