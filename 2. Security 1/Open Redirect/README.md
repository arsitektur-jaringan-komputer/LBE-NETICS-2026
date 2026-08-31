# Open Redirect

## Definisi

Open Redirect adalah kerentanan di mana penyerang dapat mengontrol tujuan client melalui redireksi pada aplikasi. Kerentanan ini disebabkan oleh kurangnya validasi terhadap parameter redireksi ("_redirection_") yang diterima oleh server, sehingga server secara sepenuhnya memercayai tujuan dari redireksi yang tertera dan mengarahkan client ke tujuan tanpa pengecekan.

erentanan ini terdaftar sebagai **CWE-601: URL Redirection to Untrusted Site ('Open Redirect')**, yang dikategorikan sebagai kelemahan di mana aplikasi web menerima input yang dikontrol pengguna sebagai tujuan redirect tanpa melakukan validasi bahwa URL tersebut mengarah ke situs yang tepercaya.

![Open Redirect](https://cwe.mitre.org/data/images/CWE-601-Diagram.png)

## Vulnerable Code

```php
// vuln.php
<?php
if (array_key_exists ("redirect", $_GET) && $_GET['redirect'] != "") {
    header ("location: " . $_GET['redirect']);
    exit;
}
http_response_code (500);
?>

<p>Missing redirect target.</p>
<?php
exit();
?>
```
 
Karena di sini kode langsung mengambil URL parameter tanpa validasi, maka ketika dilakukan fetch dengan URL parameter `redirect`, server akan mengarahkan client untuk ke tujuan yang tertera.

## Contoh Serangan

Pada contoh blok kode di atas, semisal file php tersebut dijalankan dan terdapat pada path `http://example.com/vuln.php`, 
maka contoh serangannya seperti
 
```
http://example.com/vuln.php?redirect=http://malicious.com
```

akan menyebabkan client untuk mengunjungi `http://malicious.com` alih-alih `example.com`. Skema serangan ini umumnya dimanfaatkan untuk _phishing_

Pada kasus terburuknya, Open Redirect bisa dilanjutkan menjadi kerentanan XSS karena adanya protokol `javascript:`, contoh serangannya:

```
http://example.com/vuln.php?redirect=javascript:alert(1)
```
 
akan mengeluarkan alert dari kode javascript yang di-inject di URL, ini nantinya disebut sebagai Reflected XSS.

## Secure Code

Berdasarkan rekomendasi OWASP Unvalidated Redirects and Forwards Cheat Sheet, pendekatan paling aman adalah menghindari penggunaan input pengguna secara langsung sebagai tujuan redirect, dan menggantinya dengan validasi berbasis _allow-list_ terhadap identifier atau URL yang telah ditentukan sebelumnya:
 
```php
// secure.php
<?php
// Allow-list berisi pasangan identifier => URL tujuan yang sah
$allowed_redirects = [
    "dashboard" => "/dashboard.php",
    "profile"   => "/profile.php",
    "home"      => "/index.php"
];
 
if (array_key_exists("redirect", $_GET) && array_key_exists($_GET['redirect'], $allowed_redirects)) {
    header("Location: " . $allowed_redirects[$_GET['redirect']]);
    exit;
}
 
http_response_code(400);
?>
<p>Invalid redirect target.</p>
<?php
exit;
?>
```

## Mitigasi

1. Menghindari penggunaan redirect berbasis input pengguna sepenuhnya
2. Menggunakan daftar _allow-list_ atau _whitelist_ identifier yang berisi URL relatif yang telah ditentukan oleh aplikasi
3. Membatasi redirect hanya ke URL yang relatif di domain yang sama dengan domain milik aplikasi/service
4. Tampilkan halaman konfirmasi ke pengguna sebelum melakukan redirect otomatis
5. Validasi skema URL yang dikirim oleh input pengguna dan menolak skema selain `http` ataupun `https`

## Referensi

1. https://cwe.mitre.org/data/definitions/601.html
2. https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html
3. https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/11-Client_Side_Testing/04-Testing_for_Client_Side_URL_Redirect
