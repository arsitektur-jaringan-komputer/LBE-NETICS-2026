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
```html
URL:
https://example.com/search?q=Nabil+Irawan+Top+4+WordFence

Output:
<div class="result">Hasil Pencarian dari: Nabil Irawan Top 4 WordFence</div>
```

Karena pada value dari parameter `q` tidak ada filter dan sanitasi, maka penyerang dengan mudah memanipulasinya seperti ini.

```html
URL:
https://example.com/search?q=<script>/*+Malicious+Code+Here+*/</script>

Output:
<div class="result">Hasil Pencarian dari: <script>/*+Malicious+Code+Here+*/</script></div>
```

2. Stored XSS

2. `Stored XSS`
```html
Input:
Best Alumni from BWI

Output:
<p>Hi, my name Nabil</p>
<br>
<p>Profile:</p>
<br>
<p>Best Alumni from BWI</p>
```

Sebagai contoh terdapat sebuah fitur untuk melakukan edit profile, tanpa adanya filter dan sanitasi input pada form tersebut maka penyerang bisa menyisipkan kode seperti ini.

```html
Input:
<script>/*+Malicious+Code+Here+*/</script>

Output:
<p>Hi, my name Nabil</p>
<br>
<p>Profile:</p>
<br>
<p><script>/*+Malicious+Code+Here+*/</script></p>
```

3. DOM-based XSS

```js
var urlParams = new URLSearchParams(window.location.search);
var name = urlParams.get('name');
document.getElementById('message').innerHTML = "Hello, " + name + "!";
```

Karena tidak ada filter dan sanitasi pada kolom input, maka penyerang dengan mudah menyisipkan kode berbahaya.

```html
Input:
http://example.com/page.html?name=<img src=1 onerror='/* Malicious Code Here */'>


Output:
<h1>Welcome!</h1>
<p id="message"><img src=1 onerror='/* Malicious Code Here */'></p>
```

## Mitigasi

1. Validasi semua input pengguna dengan hanya mengizinkan karakter dan format yang diharapkan
2. Menggunakan header CSP untuk membatasi resource yang dapat dijalankan oleh halaman web, contoh: hanya mengizinkan script dari sumber tertentu saja
3. Melakukan sanitasi data/input yang diterima dari pengguna sebelum diproses atau ditampilkan ke halaman web

## Referensi

1. https://portswigger.net/web-security/cross-site-scripting
2. https://owasp.org/www-community/attacks/xss/
