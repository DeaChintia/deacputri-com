---
title: 'Hosting Situs Web Personalku - Hugo Generator Situs Web Statis'
date: '2024-04-09T16:28:00+07:00'
categories: ['Web Hosting']
tags: ['Hugo', 'Web Hosting', 'Docker']
series: ['Situs Web Personal Pakai Hugo']
draft: false
---

Punya situs web personal dengan nama domain *custom* untuk menuliskan pikiran, mendokumentasikan hasil belajar, dan memamerkan proyek-proyek saya adalah *goal* yang sudah lama ingin saya lakukan. Baru ini, saya memutuskan memulai dan membeli sebuah nama domain, sekaligus berlangganan sebuah VPS, untuk meng-*hosting* situs web ini. "Kenapa VPS? Padahal ada opsi yang lebih gampang dan murah." Alasannya karena saya ingin coba proyek kecil-kecilan ini dari awal, sekalian untuk belajar.

Dalam pembuatan situs ini, saya melakukan riset kecil-kecilan untuk opsi yang dapat dipakai, mulai dari penyedia jasa layanan *cloud* (*Virtual Cloud Server*) sampai *tech stacks* yang akan dipakai. Awalnya, saya ingin pakai DigitalOcean sebagai penyedia jasa layanan *cloud* karena mereka punya masa coba gratis dengan memakai kredit terbatas. Sayang, mereka langsung menangguhkan akun saya setelah saya mendaftarkan kartu kredit. Setelah dicari-cari, rupanya mereka sering memblok akun-akun yang menggunakan kartu kredit baru sebagai upaya pencegahan *fraud*. Daripada buang-buang waktu menghubungi mereka, saya memutuskan untuk memakai penyedia jasa layanan *cloud* lokal bernama idcloudhost. Untuk *tech stacks*-nya, saya menemukan beberapa rekomendasi di internet dan memutuskan untuk menggunakan Hugo. Ini karena saya hanya butuh web statis dan Hugo sudah cukup untuk memenuhi itu.

Pada artikel pertama ini, saya akan mendokumentasikan langkah-langkah untuk meng-*hosting* situs web ini. Berikut perangkat lunak yang saya gunakan:
1. Sistem Operasi berbasis Linux untuk server web
2. Hugo sebagai generator situs web
3. Apache 2 sebagai server web
4. Git

## Sedikit tentang Hugo
Hugo adalah sebuah generator situs web statis. Hugo cukup cepat, gratis, *open source*, dan mendukung situs web banyak bahasa. Hugo juga punya banyak tema bawaan yang dapat mempercepat pembuatan web tanpa harus menyelam terlalu dalam pada sisi teknis. Kabar baiknya, Hugo mendukung penggunaan *custom template* jika seandainya saya ingin membuat templat sendiri di kemudian hari. Kalau dilihat, rasanya tidak susah menggunakan Hugo.

## Menginstal Hugo
Saya menginstal Hugo pada *VPS* Linux sebagai *production* dan pada *PC* lokal untuk *development*. Pada *VPS*, saya menginstal Hugo menggunakan *prebuilt binaries*. Untuk *PC* lokal, saya menggunakan Docker. Sebelum instal, saya membuat folder di *VPS* untuk menaruh semua *file* untuk instalasi pada direktori `home`.

```bash
# Perintah (jalankan di Terminal):
mkdir ~/installer
```

### Menginstal Prasyarat
Walaupun perangkat lunak berikut tidak dipakai semuanya di awal, saya memutuskan untuk menginstal semuanya jikalau saya butuh di kemudian hari.
1. Git
2. Go
3. Dart Sass

Sebelum mulai, saya menjalankan perintah `sudo apt update` untuk memperbarui informasi daftar *package* yang tersedia.

#### Menginstal Git
Untuk Git, saya instal menggunakan `apt install`. Lalu, hasil instalasi divalidasi dengan perintah `git version`.

```bash
# Instal Git
sudo apt install git-all

# Validasi hasil instal dengan mengecek versi Git
git version
# Keluaran
git version 2.43.0
```

#### Menginstal Go
Saya mendapatkan tautan unduhan (*download link*) *file* instalasi Go dari situs web resmi Go (https://go.dev/doc/install). Lalu, saya unduh *file* instalasi tersebut menggunakan `wget`.

```bash
# Perintah untuk mengunduh file dengan wget (jalanakn di Terminal):
wget <tautan_unduhan> -P ~/installer/

# Contoh:
wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz -P ~/installer/
```

Selanjutnya, instal go dengan perintah berikut:
```bash
# Perintah ini akan menghapus instalasi Go yang sudah ada dan menginstal package go ke /usr/local
sudo rm -rf /usr/local/go && tar -C ~/installer/ -xzf ~/installer/go1.21.1.linux-amd64.tar.gz && sudo mv ~/installer/go /usr/local/
```

Tambah baris-baris berikut ke `$HOME/.profile` atau `/etc/profile` (untuk instalasi pada seluruh user). Saya menggunakan vim sebagai editor teks. Berikut contohnya:

```bash
# /etc/profile: system-wide .profile file for the Bourne shell (sh(1))
# and Bourne compatible shells (bash(1), ksh(1), ash(1), ...).
...
...
# Applications Path    # <-- Tambahkan di akhir file
GO="/usr/local/go/bin" # <-- Tambahkan di akhir file
PATH="$PATH:$GO"       # <-- Tambahkan di akhir file
```

Terakhir, verifikasi instalasi dengan menjalankan:
```bash
# Validasi instalasi dengan mengecek versi Go
go version
# Keluaran
go version go1.21.1 linux/amd64
```

#### Menginstal Dart Sass
Buka halaman rilis (*release page*) Dart Sass, pilih versinya (saya menggunakan versi 1.66.1), dan salin tautannya. Unduh dengan `wget`:

```bash
# Perintah untuk mengunduh dengan wget (jalankan di Terminal):
wget <tautan_unduhan> -P ~/installer/

# Contoh:
wget https://github.com/sass/dart-sass/releases/download/1.66.1/dart-sass-1.66.1-linux-x64.tar.gz -P ~/installer/
```

Kemudian, ekstrak *archive file* ke direktori `~/installer/`.

```bash
# Perintah (jalankan di Terminal):
tar -xvf <nama_arsip_file> -C ~/installer/

# Contoh:
tar -xvf ~/installer/dart-sass-1.66.1-linux-x64.tar.gz -C ~/installer/
```

Pindahkan direktori `dart-sass` yang telah diekstrak ke direktori `/usr/local/`.

``` bash
# Contoh:
sudo mv ~/installer/dart-sass /usr/local/
```

Tambah baris-baris berikut ke `$HOME/.profile` atau `/etc/profile` (untuk instalasi di seluruh user). Berikut contohnya:

```bash
# /etc/profile: system-wide .profile file for the Bourne shell (sh(1))
# and Bourne compatible shells (bash(1), ksh(1), ash(1), ...).
...
...
# Applications Path
DART_SASS="/usr/local/dart-sass" # <-- Tambahkan di sini
GO="/usr/local/go/bin"
PATH="$PATH:$DART_SASS:$GO" # <--Ubah baris ini
```

Akhiri dengan memverifikasi instalasi:
```bash
# Memvalidasi Instalasi dengan Memeriksa Versi Sass
sass --version
# Keluaran
1.66.1
```

### Menginstal Hugo - Instalasi dengan *Prebuild Binaries* di Linux
1. Buka situs web https://github.com/gohugoio/hugo/tags
2. Unduh versi yang akan digunakan. Untuk situs web saya, saya menggunakan Hugo versi 0.119.0.

    ```bash
    # Perintah (jalankan di Terminal):
    wget <tautan_unduhan> -P ~/installer/
    
    # Contoh:
    wget https://github.com/gohugoio/hugo/releases/download/v0.119.0/hugo_extended_0.119.0_Linux-64bit.tar.gz -P ~/installer/
    ```

3. Ekstrak *archive file*

    ```bash
    # Perintah (jalankan di Terminal):
    tar -xvf <archive_file_name> -C ~/installer/

    # Contoh:
    tar -xvf ~/installer/hugo_extended_0.119.0_Linux-64bit.tar.gz -C ~/installer/
    ```

4. Pindahkan *executable file* "hugo" ke `/usr/local/bin/`. Direktori ini sudah ada pada *variable environment* PATH Linux, jadi tidak perlu menambahkannya lagi nanti.

    ```bash
    # Perintah (jalankan di Terminal):
    sudo mv hugo /usr/local/bin/
    ```

5. Jalankan perintah ini untuk mengkonfirmasi bahwa Hugo telah terinstal dengan benar.

    ```bash
    # Perintah (jalankan di Terminal):
    hugo env
    # Keluaran:
    hugo v0.119.0-b84644c008e0dc2c4b67bd69cccf87a41a03937e+extended linux/amd64 BuildDate=2023-09-24T15:20:17Z VendorInfo=gohugoio
    GOOS="linux"
    GOARCH="amd64"
    GOVERSION="go1.21.1"
    github.com/sass/libsass="3.6.5"
    github.com/webmproject/libwebp="v1.2.4"
    github.com/sass/dart-sass/protocol="2.1.0"
    github.com/sass/dart-sass/compiler="1.66.1"
    github.com/sass/dart-sass/implementation="1.66.1"
    ```

### Menginstal Hugo - Konfigurasi Docker *Container* pada Windows
Untuk *development*, saya menggunakan Docker *container* di mesin Windows saya. Pertama, saya menginstal Docker Desktop di Windows, mengikuti [dokumentasi resmi Docker](https://docs.docker.com/desktop/install/windows-install/) untuk ini. Kemudian, saya membuat *custom Docker image* menggunakan Dockerfile ini:


#### Menyiapkan Dockerfile

```dockerfile
# Gunakan *base image*
FROM ubuntu:24.04 AS base

WORKDIR /tmp

# Perbarui *package index file* dan install semua *package* yang dibutuhkan
RUN apt-get update && apt-get install -y git wget

# Instal Go
RUN wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz -P /tmp/ && \
    tar -C /usr/local -xzf /tmp/go1.21.1.linux-amd64.tar.gz && \
    rm /tmp/go1.21.1.linux-amd64.tar.gz

# Instal Hugo
RUN wget https://github.com/gohugoio/hugo/releases/download/v0.119.0/hugo_extended_0.119.0_linux-amd64.tar.gz -P /tmp/ && \
    tar -C /usr/local/bin -xzf /tmp/hugo_extended_0.119.0_linux-amd64.tar.gz && \
    rm /tmp/hugo_extended_0.119.0_linux-amd64.tar.gz

# Instal Dart Sass
RUN wget https://github.com/sass/dart-sass/releases/download/1.66.1/dart-sass-1.66.1-linux-x64.tar.gz -P /tmp/ && \
    tar -C /usr/local -xzf /tmp/dart-sass-1.66.1-linux-x64.tar.gz && \
    rm /tmp/dart-sass-1.66.1-linux-x64.tar.gz

# Atur PATH untuk Go & Dart Sass
ENV PATH="$PATH:/usr/local/go/bin:/usr/local/dart-sass"

# Mengekspos port untuk server Hugo
EXPOSE 1313
```

Dockerfile yang disediakan di atas akan membangun (*build*) *Docker image* dengan spesifikasi berikut:
- Ubuntu 24.04 sebagai sistem operasi
- go 1.21.1
- hugo extended 0.119.0
- dart-sass 1.66.1

Versi semua perangkat lunak yang terdaftar dapat diubah berdasarkan kebutuhan dengan memodifikasi tautan unduhan perangkat lunak di dalam Dockerfile.

Terakhir, bangun *Docker image* dengan perintah ini::
```docker build -t de-hugo-dev .```

#### Menyiapkan *Docker Compose File*
*Docker compose file* ini akan digunakan untuk menjalankan kontainer Docker dan melayani situs web dengan `hugo server`, menggunakan proyek yang terletak di `./srv/http/src`.

```yaml
services:
  hugo-server:
    image: "de-hugo-dev"
    ports:
      - "1313:1313"
    container_name: "de-hugo-dev"
    entrypoint: hugo
    command: "server -D --bind 0.0.0.0 --port 1313 --poll 700ms"
    working_dir: /srv/http/src
    volumes:
      - "./srv/http/src:/srv/http/src"
```

Catatan:
- `--poll 700ms` ditambahkan untuk mengatasi [**Masalah 1**](#masalah-1-hugo-tidak-memperbarui-situs-setelah-perubahan-file-di-host)
## Menjalankan Hugo di *Development*
### Membuat Proyek Baru
Untuk membuat proyek Hugo baru dengan Docker, saya menjalankan perintah berikut:
```bash
docker run --rm --name de-hugo-dev -p 1313:1313 -v "./srv/http/src:/srv/http/src" --entrypoint "hugo" de-hugo-dev new site . --force
```

Perintah di atas akan membuat sebuah proyek Hugo baru dan menaruhnya di dalam `./srv/http/src/` pada mesin *host* saya. Kontainer kemudian akan segera dihentikan dan dihapus. Perintah di atas hanya akan digunakan untuk membuat proyek baru saja. Untuk proyek yang sudah ada, kontainer akan dibuat dan dijalankan dengan `docker compose`.

### Menjalankan Kontainer Docker dan Mengaktifkan Situs Web
Untuk menjalankan kontainer Docker, jalankan:
```bash
docker compose up -d
```

Perintah itu akan menjalankan situs pengembangan (*development*). Setiap perubahan yang dilakukan di folder `./srv/http/src` akan tercermin secara langsung di situs web (http://localhost:1313).

## Menerbitkan Situs Web di *Production*
Ubah *file* konfigurasi Hugo (`config.yml`, `hugo.yaml`, `hugo.toml`, atau `hugo.json` - saya lebih suka menggunakan `config.yml`) dan lakukan perubahan berikut ini: 
1. Tetapkan baseURL untuk situs *production* dengan menggunakan nama domain VPS.
2. Tetapkan judul situs *production*.
3. Tetapkan wilayah dan bahasa yang diinginkan.   

```yaml
baseURL: "http://deacputri.com"
languageCode: "en-us"
title: De's Personal Site
```

Untuk mempublikasikan situs, jalankan:
```hugo```

Perintah ini akan menghasilkan situs yang siap dipublikasikan dalam direktori `public`. Kemudian, pada VPS, buat direktori `/srv/http/nama-situs-web-anda`. Salin folder `public` yang dihasilkan dari *development* ke `/srv/http/nama-situs-web-anda` di *production*.

Selanjutnya, ubah *file* konfigurasi Apache2:
```bash
sudo vi /etc/apache2/apache2.conf
```

Jadikan baris-baris berikut sebagai komentar:
```xml
# <Directory /var/www/>
#         Options Indexes FollowSymLinks
#         AllowOverride None
#         Require all granted
# </Directory>
```

dan tambahkan baris-baris berikut::
```xml
<Directory /srv/http/your-website-name/public/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```

Simpan perubahannya.

Kemudian, ubah *file* konfigurasi *default virtual host*:
```bash
sudo vi /etc/apache2/sites-available/000-default.conf
```

Ubah baris-baris ini dari:
```xml
<VirtualHost *:80>
    ...
    ...
    DocumentRoot /var/www/html
    ...
    ...
</VirtualHost>
```

menjadi:
```xml
<VirtualHost *:80>
    ...
    ...
    DocumentRoot /srv/http/your-website-name/public
    ...
    ...
</VirtualHost>
```

Terakhir, mulai ulang (*restart*) layanan server web:
```bash
sudo service apache2 restart
```

Akses situs web dengan menggunakan alamat IP global atau domain name yang sudah dikonfigurasi pada *VPS*. Perlu dicatat bahwa server web akan menampilkan "*Page Not Found*" karena proyek Hugo masih kosong.

## Memperbaiki Masalah

### Masalah 1 Hugo Tidak Memperbarui Situs Setelah Perubahan File di Host
<table>
    <tr>
        <td>Masalah</td>
        <td>
            Ketika mengedit <i>file</i> pada <i>development</i> di <i>host</i>, hal yang diharapkan adalah perubahan langsung terlihat di http://localhost:1313. Namun, walau perubahan <i>file</i> di <i>host</i> berhasil tercermin di dalam <i>container</i>, Hugo tidak memperbarui situs web ketika terjadi perubahan.
        </td>
    </tr>
    <tr>
        <td>Penyebab</td>
        <td>
            Masalah yang umum terjadi pada docker yang di-<i>host</i> di Windows. Berdasarkan <a href="https://github.com/klakegg/docker-hugo/issues/61#issuecomment-1198441388">sumber berikut</a>:
            <blockquote cite="https://github.com/klakegg/docker-hugo/issues/61#issuecomment-1198441388">
            "<i>In cases where Hugo CLI is running in a shared directory on linux VM on a windows host the dev server is not detecting file changes from the host environment. (ex: Whenever a docker dev environment is running on a windows host.). This is because by default most watchers will use native file system change detection. And it is not working well in this shared situation.</i>"
            </blockquote>
        </td>
    </tr>
    <tr>
        <td>Solusi</td>
        <td>
            Menambahkan <code>--poll 700ms</code> pada perintah di docker <i>compose file</i> saat menjalankan "hugo server".
        </td>
    </tr>
</table>
