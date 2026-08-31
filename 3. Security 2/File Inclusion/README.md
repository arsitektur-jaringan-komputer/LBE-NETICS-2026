# File Inclusion

## Definisi

File Inclusion atau paling umum dikenal dalam bentuk LFI (Local File Inclusion) adalah jenis serangan yang mengeksploitasi aplikasi untuk menyertakan file lokal dari sistem server. File yang disertakan mungkin file sensitif seperti file konfigurasi, file log, atau file sistem lain yang seharusnya tidak diakses pengguna.

File yang berhasil disertakan melalui LFI dapat berupa file sensitif seperti file konfigurasi aplikasi (berisi kredensial database), file log server, source code aplikasi lain, hingga file sistem operasi seperti `/etc/passwd` pada Linux.

Kerentanan ini berbeda dengan _Remote File Inclusion (RFI)_, di mana penyerang menyertakan file dari server eksternal (URL), bukan dari sistem lokal server target.

![LFI](https://miro.medium.com/v2/resize:fit:644/1*UPMlwBWgKMSUzSvY5mt5uw.png)

## Vulnerable Code

```php
<?php
if (isset($_GET['page'])) {
    $page = $_GET['page'];
    include($page);
} else {
    echo "Page not found.";
}
?>
```

Dalam kode blok diatas, tidak adanya validasi maupun sanitasi terhadap input parameter GET `page` yang diberikan pengguna, 
nilai tersebut langsung digunakan sebagai argumen pada fungsi `include()`, hal ini menyebabkan kerentanan LFI.


## Contoh Serangan

```
# Request

http://localhost/vulnerabilities/fi/?page=file2.php

# Response Normal

"I needed a password eight characters long so I picked Snow White and the Seven Dwarves." ~ Nick Helm


# Request

http://localhost/vulnerabilities/fi/?page=/etc/passwd

# Response

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/bin/false
mysql:x:101:101:MySQL Server,,,:/nonexistent:/bin/false

(hanya contoh)
```

Pada request pertama menunjukkan penggunaan normal aplikasi, di mana `page=file2.php` meng-include file PHP yang memang disediakan oleh aplikasi. 

Pada request kedua, parameter `page` diganti dengan path absolut `/etc/passwd`, sehingga fungsi `include()` menyertakan isi file sistem tersebut dan menampilkannya sebagai bagian dari response,
yang dimana seharusnya tidak diperbolehkan

## Secure Code

```php
<?php
$whitelist = ['home.php', 'about.php', 'contact.php'];
if (isset($_GET['page']) && in_array($_GET['page'], $whitelist)) {
    include($_GET['page']);
} else {
    echo "Page not found.";
}
```

Nilai parameter `page` dari pengguna dicocokkan terlebih dahulu terhadap daftar tersebut menggunakan `in_array()` sebelum diproses oleh `include()`

Jika nilai parameter tidak ada dalam array variabel whitelist, maka web hanya akan menampilkan `"Page Not Found"`.

## Mitigasi

1. Melakukan validasi dan sanitasi input pengguna
2. Menggunakan whitelist untuk membatasi file-file yang dapat di akses
3. Menghindari penggunaan input pengguna secara langsung ke fungsi-fungsi pemanggilan file seperti `include`, `require` dan sejenisnya
4. Menerapkan prinsip least privilege dalam menjalankan aplikasi / service

## Referensi

1. https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion
2. https://portswigger.net/web-security/file-path-traversal

