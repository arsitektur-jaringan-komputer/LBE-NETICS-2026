# Docker Setup & Installation

## Pre-requisite

## Instalasi Docker

### Docker Desktop

Docker Desktop adalah aplikasi yang memudahkan instalasi Docker di Windows dan macOS. 
Docker Desktop sudah mencakup Docker Engine, Docker CLI, dan Docker Compose dalam satu paket instalasi, sehingga pengguna tidak perlu menginstal komponen-komponen tersebut secara terpisah.

### Docker Engine

Docker Engine adalah komponen inti (core) dari Docker yang bertugas menjalankan dan mengelola container. 
Pada sistem operasi Linux, Docker Engine umumnya diinstal secara terpisah (tanpa GUI) melalui package manager masing-masing distro.

### Docker Compose

Docker Compose adalah tool untuk mendefinisikan dan menjalankan aplikasi Docker multi-container menggunakan file konfigurasi (biasanya `docker-compose.yml`). 
Pada Docker Desktop, Compose sudah termasuk secara otomatis; sedangkan pada instalasi Docker Engine di Linux, Compose perlu diinstal sebagai plugin tambahan (`docker-compose-plugin`).

#### Windows

1. Pastikan bahwa WSL2 sudah terinstall, jika belum, ikuti langkah-langkah di https://pureinfotech.com/install-windows-subsystem-linux-2-windows-10/ (cek versi Windows 10 anda terlebih dahulu; jika versi 2004 ke atas termasuk Windows 11, ikuti langkah-langkah di atas, jika versi 1909 ke bawah, scroll ke bawah pada halaman tersebut).
2. Download installer Docker Desktop di [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop) (ukuran ±490 MB). Docker Desktop sudah include Docker Engine dan Docker Compose.
3. Jalankan installernya, lalu pencet ok/install.
4. Docker sudah terinstall.

> Jika muncul peringatan `WSL 2 requires an update to its kernel component.` ketika aplikasi dijalankan, 
> download link berikut: [https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi), 
> jalankan setup wizard yang sudah didownload, kemudian buka kembali aplikasi Docker.
> atau mengikuti instruksi dari stackoverflow berikut : [https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

#### Linux

Distro-distro Linux yang tercantum dalam dokumentasi official docker [https://docs.docker.com](https://docs.docker.com)

##### CentOS

0. Prasyarat:
   - CentOS 7, 8, atau 9
   - Repo **`centos-extras`** harus sudah diaktifkan
1. Siapkan repo
```
    sudo yum install -y yum-utils
    sudo yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
```
2. Install Docker Engine, containerd, dan Docker Compose
```
    sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
3. Mulai Docker
```
    sudo systemctl start docker
```
4. Pastikan Docker berjalan sempurna
```
    sudo docker run hello-world
```

##### Debian

Termasuk Kali Linux, MX Linux, Ubuntu
 
1. Kebutuhan sistem minimal: Debian Buster 10 _64-bit_
2. Update package apt lalu install package berikut agar apt bisa menggunakan repository https
```
    sudo apt-get update
 
    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg
```
3. Tambahkan kunci GPG Docker
```
    sudo install -m 0755 -d /etc/apt/keyrings
 
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
 
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
4. Gunakan command berikut untuk memilih repo stabil
```
    echo \
      "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
      "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
5. Install Docker Engine dan Docker Compose
```
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
6. Pastikan Docker sudah terinstall dengan benar
```
    sudo docker run hello-world
```

##### Fedora

Disini diasumsikan menggunakan Fedora Workstation dan bukan server (jika anda sudah mahir dengan CLI seharusnya anda tidak butuh tutorial). Maka kami akan menyarankan anda menginstall Docker Desktop, yang sudah include Docker Engine dan Docker Compose.
 
1. Prasyarat:
   - Gunakan versi 64-bit dari Fedora 36 atau 37 (bukan 38)
   - Jika anda menggunakan desktop environment Gnome, anda harus menginstall ekstensi AppIndicator and KStatusNotifierItem
   - Untuk DE non-Gnome, anda harus menginstall gnome-terminal
2. Siapkan Docker Package Repository
```bash
    sudo dnf -y install dnf-plugins-core
    sudo dnf config-manager \
        --add-repo \
        https://download.docker.com/linux/fedora/docker-ce.repo
```
3. Download package RPM Docker Desktop di link berikut: https://docs.docker.com/desktop/install/fedora/
4. Install sesuai dengan versi yang didownload (gunakan autocomplete/tab agar lebih mudah)
```
    sudo dnf install ./docker-desktop-<version>-<arch>.rpm
```
5. Buka aplikasi Docker Desktop dari menu aplikasi. Docker Desktop akan otomatis menjalankan daemon (Docker Engine) di background.
6. Pastikan daemon aktif dan bisa diakses dari terminal
```
    docker info
```
7. Pastikan Docker sudah terinstall dengan benar
```
    sudo docker run hello-world
```
 
> **Catatan:** Jika sebelumnya kalian juga sudah menginstall Docker Engine native (bukan Desktop) di Fedora yang sama, kedua daemon dapat coexist selama kalian mengatur context yang tepat dengan `docker context ls` dan `docker context use <name>` agar tidak terjadi konflik socket.
 
##### RHEL

Instalasi Docker Engine pada Red Hat Enterprise Linux (RHEL) menggunakan mekanisme repository yang serupa dengan CentOS, karena keduanya berbasis RPM/yum-dnf.
 
1. Siapkan repo (perhatikan: RHEL memiliki URL repo tersendiri, berbeda dari CentOS)
```
    sudo yum install -y yum-utils
    sudo yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/rhel/docker-ce.repo
```
2. Install Docker Engine, containerd, dan Docker Compose
```
    sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
    > Jika muncul prompt GPG key, pastikan fingerprint sesuai dengan `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35` sebelum menerimanya.
3. Mulai Docker daemon dan aktifkan agar berjalan otomatis setiap boot
```
    sudo systemctl start docker
    sudo systemctl enable docker
```
4. Pastikan Docker berjalan sempurna
```
    sudo docker run hello-world
```
 
> Catatan: pada beberapa versi RHEL (mis. RHEL 8/9), paket `docker` bawaan sudah digantikan oleh `podman`. Jika ada konflik, hapus `podman` dan `buildah` terlebih dahulu sebelum instalasi Docker.
>
> Untuk detail dan opsi instalasi terbaru khusus RHEL, silakan cek halaman resmi: [Install Docker Engine on RHEL](https://docs.docker.com/engine/install/rhel/)

##### Ubuntu

1. Kebutuhan sistem minimal: Ubuntu 18.04 (LTS) _64-bit_
2. Update package apt lalu install package berikut agar apt bisa menggunakan repository https
```
    sudo apt-get update
 
    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg
```
3. Tambahkan kunci GPG Docker
```
    sudo install -m 0755 -d /etc/apt/keyrings
 
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
 
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
4. Gunakan command berikut untuk memilih repo stabil
```
    echo \
      "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
5. Install Docker Engine dan Docker Compose
```
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
6. Mulai Docker daemon dan aktifkan agar berjalan otomatis setiap boot
```
    sudo systemctl start docker
    sudo systemctl enable docker
```
7. Pastikan Docker sudah terinstall dengan benar
```
    sudo docker run hello-world
```

##### Arch

1. Install docker dan docker-compose
    ```
    pacman -Syu docker docker-compose
    ```
2. Jalankan docker service
    ```
    systemctl start docker.service
    ```
3. Cek apakah docker sudah berjalan
    ```
    docker info
    ```
4. Pastikan anda bisa menjalankan container
    ```
    docker run -it --rm archlinux bash -c "echo hello world"
    ```

##### Ubuntu

Untuk instalasi di distro lain bisa mengunjungi website official docker di [Docker Docs](https://docs.docker.com)

#### MacOS

Kebutuhan sistem minimal: macOS versi 11 dengan RAM 4 GB_
 
##### GUI
 
1. Download installer melalui link berikut: https://www.docker.com/products/docker-desktop
2. Jalankan installernya, kemudian drag ikon Docker menuju ikon folder _Application_
3. Jalankan aplikasinya dari Launchpad atau folder _Application_
4. Jika muncul peringatan "Are you sure you want to open it?", tekan open
5. Baca terms and condition dan tekan accept
6. Pilih recommended setting dan tekan ok
7. Masukkan password mac dan tunggu hingga proses selesai
8. Docker Desktop akan otomatis menjalankan daemon (Docker Engine) di background setiap kali aplikasi dibuka. Pastikan daemon aktif dengan membuka Terminal dan menjalankan:
```
    docker info
```

Jika muncul detail server (bukan error `Cannot connect to the Docker daemon`), daemon sudah berjalan alongside Docker Desktop.

##### Terminal

1. Cek apakah Homebrew sudah terinstall
```
    brew --version
```
2. Jika belum, install Homebrew terlebih dahulu
```
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```
3. Install Docker
```
    brew install --cask docker
```
4. Jalankan Docker
```
    open /Applications/Docker.app
```
5. Jika muncul peringatan "Are you sure you want to open it?", tekan open
6. Baca terms and condition dan tekan accept
7. Pilih recommended setting dan tekan ok
8. Masukkan password mac dan tunggu hingga proses selesai
9. Pastikan daemon aktif (Docker Desktop harus dalam keadaan terbuka/running agar daemon jalan):

```
    docker info
```

### Test Docker dengan Hello World

Setelah proses instalasi selesai (baik melalui Docker Desktop maupun Docker Engine di Linux), pastikan Docker berjalan dengan benar menggunakan image `hello-world`:
 
```
docker run hello-world
```

Jika instalasi berhasil, akan muncul pesan konfirmasi seperti gambar di bawah:
![Docker Hello World](./assets) 

## Referensi

1. [Docker Official Website](https://www.docker.com)
2. [Docker Manuals Official Website](https://docs.docker.com/manuals/)
3. [Get Started with Docker Desktop](https://www.docker.com/products/docker-desktop)
4. [Install Docker Engine](https://docs.docker.com/engine/install/)
5. [Install Docker Compose](https://docs.docker.com/compose/install/)
<!-- - [Pelatihan Linux Repository](https://github.com/arsitektur-jaringan-komputer/Pelatihan-Linux) -->
