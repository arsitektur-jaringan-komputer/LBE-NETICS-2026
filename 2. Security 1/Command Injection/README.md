# Command Injection

## Definisi

Command Injection adalah jenis kerentanan di mana seorang penyerang dapat menjalankan perintah yang tidak sah pada sistem host yang menjadi target. Kerentanan ini biasanya terjadi pada aplikasi yang secara tidak aman memproses/validasi/sanitasi input pengguna dan kemudian menggunakan input tersebut sebagai bagian dari perintah sistem atau shell.

Ketika input pengguna digabungkan langsung ke dalam perintah sistem tanpa adanya validasi/sanitasi melalui fungsi-fungsi seperti pada PHP (`shell_exec`, `system`, `exec` ,dll.), penyerang dapat langsung menyisipkan karakter khusus shell seperti `&&`, `;`, `||`, `|` atau backtick ``` untuk menambahkan command tambahan di luar command yang dimaksudkan oleh developer.

## Contoh Vulnerable Code

```php
<?php
if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}
?>
```

Kode di atas merupakan fitur untuk melakukan ping ke suatu alamat IP/host yang dimasukkan oleh pengguna melalui parameter `ip`. 
Nilai `$target` yang berasal langsung dari input pengguna (`$_REQUEST['ip']`) digabungkan langsung ke dalam perintah `ping` menggunakan `shell_exec()` tanpa adanya proses validasi maupun sanitasi input sama sekali.
 
Karena tidak ada filter terhadap karakter khusus shell, penyerang dapat menyisipkan perintah tambahan pada parameter `ip`, misalnya:
 
```
127.0.0.1 && whoami
127.0.0.1; cat /etc/passwd
127.0.0.1 | id
```

Dengan contoh input seperti pada blok diatas, perintah ping akan tetap dijalankan, namun perintah tambahan yang disisipkan setelah karakter khusus shell juga akan tereksekusi.

## Secure Code

```php
<?php
if( isset( $_POST[ 'Submit' ] ) ) {

    $target = filter_var($_REQUEST['ip'], FILTER_VALIDATE_IP);
    if ($target === false) {
        die('Alamat IP tidak valid.');
    }

    if (stristr(PHP_OS, 'WIN')) {
        // Windows
        $command = escapeshellcmd('ping ' . $target);
    } else {
        // *nix
        $command = escapeshellcmd('ping -c 4 ' . $target);
    }

    $output = shell_exec($command);

    // Feedback untuk pengguna akhir
    echo "<pre>" . $output . "</pre>";
}
?>
```

Penjelasan kode diatas sebagai berikut:

- `$target = filter_var($_REQUEST['ip'], FILTER_VALIDATE_IP);`: Filter dan validasi terhadap IP.
- `escapeshellcmd`: Mencegah karakter berbahaya.

## Mitigasi

1. Hindari penggunaan fungsi yang menjalankan perintah shell secara langsung
2. Validasi input pengguna dengan whitelist
3. Menggunakan fungsi string escape yang sesuai
4. Sebisa mungkin menggunakan parametized API/library (fungsi yang menerima argumen secara terpisah-pisah)
5. Menerapkan prinsip least-privilege, memastikan user yang digunakan oleh service memiliki user dengan hak akses minimal

## Referensi

1. [Portswigger - OS Command Injection](https://portswigger.net/web-security/os-command-injection)
2. [OWASp - Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
