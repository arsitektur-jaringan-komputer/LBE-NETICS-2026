# File Upload

## Definisi

File Upload merupakan celah kerentanan yang dimana web server memperbolehkan user untuk mengunggah file tanpa adanya validasi seperti nama file, tipe file, konten file, dan ukuran file.

File Upload terjadi karena pada web server tidak adanya pembatasan file yang akan diupload oleh user, disisi lain pihak developer merasa fiturnya aman yang sebenarnya dapat mudah untuk dilakukan bypass.

![file-upload](https://www.cobalt.io/hs-fs/hubfs/file-upload-vulnerabilities-example.png)

## Vulnerable Code

```php
<?php
if (isset($_FILES['file'])) {
    $target_dir = "uploads/";
    $target_file = $target_dir . basename($_FILES["file"]["name"]);
    if (move_uploaded_file($_FILES["file"]["tmp_name"], $target_file)) {
        echo "The file ". basename($_FILES["file"]["name"]). " has been uploaded.";
    } else {
        echo "Sorry, there was an error uploading your file.";
    }
}
?>
```

Di blok kode diatas tidak dilakukan validasi terhadap jenis file dan ukuran file, sehingga rentan terhadap upload file yang berbahaya. (seperti web shell) yang dapat langsung dieksekusi oleh server.
 
![chain-attack](https://pbs.twimg.com/media/F-FZQHabYAAqUhW?format=jpg&name=large)


## Contoh Serangan

1. Menggunakan double extension, memanfaatkan kesalahan konfigurasi server yang hanya memeriksa apakah nama file mengandung ekstensi tertentu tanpa memvalidasi ekstensi terakhir yang benar-benar dieksekusi:

```
file.jpg.php
```
 
2. Menggunakan null byte untuk memotong pengecekan ekstensi pada implementasi lama yang rentan terhadap null byte injection, sehingga server membaca file sebagai `file.php` meskipun tervalidasi sebagai `.gif`:

```
file.php%00.gif
```
 
3. Manipulasi konten dengan menyisipkan magic bytes gambar (`GIF89a;`) di awal file agar lolos validasi berbasis pembacaan header sederhana, sementara isi file tetap berupa payload PHP:

```
POST /images/upload/ HTTP/1.1
Host: target.com
...
---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="setya404.php"
Content-Type: image/gif
 
GIF89a; <?php system("id") ?>
```

## Secure Code

```php
<?php
if (isset($_FILES['file'])) {
    $allowed_types = array('jpg', 'jpeg', 'png');
    $allowed_mime_types = array('image/jpeg', 'image/png');
    $max_size = 2 * 1024 * 1024;
    $upload_dir = "uploads/";
    $file_name = basename($_FILES["file"]["name"]);
    $file_size = $_FILES["file"]["size"];
    $file_tmp = $_FILES["file"]["tmp_name"];
    $file_ext = strtolower(pathinfo($file_name, PATHINFO_EXTENSION));
    $file_mime = mime_content_type($file_tmp);
 
    if (!in_array($file_ext, $allowed_types)) {
        echo "File extension is not allowed.";
        exit();
    }
    if (!in_array($file_mime, $allowed_mime_types)) {
        echo "File MIME type is not allowed.";
        exit();
    }
    if ($file_size > $max_size) {
        echo "File size exceeds the limit.";
        exit();
    }
    if (in_array($file_ext, array('jpg', 'jpeg', 'png', 'gif'))) {
        $image_info = getimagesize($file_tmp);
        if ($image_info === false) {
            echo "File header is invalid or not a real image.";
            exit();
        }
    }
 
    $new_file_name = uniqid() . "." . $file_ext;
    if (move_uploaded_file($file_tmp, $upload_dir . $new_file_name)) {
        echo "File uploaded successfully!";
    } else {
        echo "Error uploading file.";
    }
}
?>
```

Pada kode diatas, file yang diupload akan di validasi dan hanya memperbolehkan ekstensi tertentu yang dapat disimpan ke server.

Selain itu, juga melakukan validasi dari MIME type dengan menggunakan fungsi `mime_content_type` untuk membaca magic byte yang tersimpan dalam file yang dikirim.

Dan beberapa lapisan keamanan lainnya untuk validasi dan sanitasi, seperti validasi header file (dalam kasus ini gambar) dan penggantian nama file menjadi nama yang acak.

## Mitigasi

1. Validasi ekstensi dan tipe MIME file menggunakan _allow-list_ untuk membatasi tipe dan ekstensi file yang diperbolehkan.
2. Menggunakan nama acak untuk penamaan file yang di-generate oleh server (contoh pada kode menggunakan fungsi `uniqid()`)
3. Membatasi ukuran file yang boleh di upload oleh user dengan mengecek ukuran tiap file yang diupload.
4. Melakukan validasi tipe file, header file dan isi dari konten file yang diupload
5. Menerapkan CSP dan header `X-Content-Type-Options: nosniff` pada direktori penyimpanan uploaded file untuk mencegah eksekusi file yang di-serve dengan tipe konten yang tidak sesuai

## Referensi

1. https://portswigger.net/web-security/file-upload
2. https://cyberw1ng.medium.com/understanding-file-upload-vulnerabilities-in-web-app-penetration-testing-6de583fba63f
3. https://github.com/daffainfo/AllAboutBugBounty/blob/master/Arbitrary%20File%20Upload.md
