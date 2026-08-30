# Content Security Policy (CSP)

## Definisi

Content Security Policy (CSP) merupakan fitur berupa HTTP Header yang dapat digunakan untuk menghindari berbagai ancaman keamanan. Biasanya, CSP digunakan sebagai proteksi terhadap serangan cross-site scripting (XSS), data injection, dan clickjacking dengan cara mengontrol content sources yang diakses oleh web application. Namun, jika pengimplementasian CSP tidak ketat maka dapat di-bypass. 

CSP umumnya digunakan sebagai lapisan pertahanan tambahan (defense in-depth) terhadap serangan seperti:
- Cross-Site Scripting (XSS)
- Data Injection Attack]
- Clickjacking

Efektivitas penerapan CSP terhadap keamanan aplikasi bergantung pada seberapa ketat kebijakan yang diterapkan.
Konfigurasi CSP yang terlalu longgar—misalnya mengizinkan `unsafe-inline`, `unsafe-eval`, wildcard (`*`), atau domain eksternal yang dapat disalahgunakan sebagai host konten publik (seperti Pastebin, Google Docs, atau CDN yang mengizinkan upload bebas) dapat menyebabkan CSP tersebut di-bypass oleh penyerang.

## Contoh Vulnerable Code

```php
header("Content-Security-Policy: script-src 'self' https://pastebin.com;");

if (isset($_POST['include'])) {
    $page['body'] .= "<script src='" . $_POST['include'] . "'></script>";
}

$page['body'] .= '
<form name="csp" method="POST">
    <p>You can include scripts from external sources, examine the Content Security Policy and enter a URL to include here:</p>
    <input size="50" type="text" name="include" value="" id="include" />
    <input type="submit" value="Include" />
</form>';
?>
```

Penjelasan header CSP

Header CSP pada blok kode di atas menunjukkan definisi directive `script-src` dengan nilai `'self' https://pastebin.com` yang berarti browser hanya diizinkan memuat dan mengeksekusi script yang berasal ddari URL/origin website itu sendir (self) atau `pastebin.com`

## Contoh Serangan

Dari vulnerable code di atas, serangan dapat dilakukan dengan memanfaatkan domain eksternal (`https://pastebin.com`) yang telah diizinkan sebagai `script-src`. Penyerang cukup membuat sebuah paste baru berisi payload JavaScript, misalnya:
 
```js
alert('CSP VULNERABLE!')
```

Karena `pastebin.com` termasuk dalam whitelist `script-src` pada header CSP, browser tidak akan memblokir permintaan tersebut, script akan berhasil dimuat dan dieksekusi, 
sehingga pop-up alert dengan tulisan `CSP VULNERABLE!` akan muncul—membuktikan bahwa CSP yang diterapkan berhasil di-bypass melalui domain eksternal yang seharusnya tidak dipercaya secara penuh.

## Mitigasi

1. Hindari whitelist domain publik/pihak ketiga yang tidak sepenuhnya terpercaya
2. Mengimplementasikan strict CSP menggunakan notice-based atau hash-based directive

contoh: 
```
  Content-Security-Policy: script-src 'nonce-{random-value}' 'strict-dynamic';
```

3. Hindari penggunaan `unsafe-inline` atau `unsafe-eval` pada directive `script-src` (membuka celah eksekusi inline script)
4. Gunakan `base-uri 'self'` untuk mencegah manipulasi tag `<base>` yang dapat digunakan untuk mengubah base URL resolusi resource relatif pada halaman.
5. Audit dan uji CSP secara berkala

## Referensi

1. https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP
2. https://portswigger.net/web-security/cross-site-scripting/content-security-policy
