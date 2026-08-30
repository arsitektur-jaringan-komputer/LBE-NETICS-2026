# Brute Force

## Definisi

Bruteforce attack merupakan salah satu serangan siber yang dimana seorang penyerang mencoba menebak suatu informasi seperti kata sandi, kunci enkripsi, dll.

Serangan ini dinamakan "brute force" karena mengandalkan kekuatan komputasi dan mencoba berbagai kombinasi secara sistematis.

Brute Force dapat menyasar berbagai target, seperti Login Form pada aplikasi web atau layanan seperti FTP, SSH dll.; Hash Password dari data yang bocor; direktori atau file tersembunyi.

## Jenis

1. Simple Bruteforce: Menggunakan semua kombinasi yang mungkin tanpa ada pola tertentu.
2. Dictionary Attack: Menggunakan daftar kata atau frasa (wordlist) yang sering digunakan.
3. Hybrid Attack: Kombinasi antara Dictionary Attack dan Simple Bruteforce, misalnya menggunakan kombinasi kata dengan tambahan angka atau simbol.
4. Credential Stuffing: Menggunakan kombinasi username dan password yang telah bocor dari layanan lain (data breach) untuk dicoba pada layanan target, memanfaatkan kebiasaan pengguna yang menggunakan kredensial sama di berbagai platform.

dan lain sebagainya.

## Tools

| Image | Tools | Usage |
|----|----------|-----------|
|![Hashcat](https://media.licdn.com/dms/image/D5612AQHYUVori41ImQ/article-cover_image-shrink_600_2000/0/1703038353884?e=2147483647&v=beta&t=5eNfTWjtJDb-VYPCJ6sQjgA0PtkOS4F-9uenJSYI-bA) | [Hashcat](https://hashcat.net/hashcat/) | Alat untuk melakukan hash cracking dengan mencoba berbagai jenis serangan seperti dictionary attack atau hybrid attack untuk menambahkan variasi dan kompleksitas. | 
|![Hydra](https://miro.medium.com/v2/resize:fit:1400/1*ZmsTZcYyMpoIOIWCbTPOPA.png) | [Hydra](https://www.kali.org/tools/hydra/) | Alat yang digunakan untuk melakukan bruteforce dengan dukungan berbagai protokol seperti HTTP, FTP, SMB, SSH, dan lainnya.    |
|![Dirsearch](https://assets.offsec.tools/tools/dirsearch-3702.png)| [Dirsearch](https://github.com/maurosoria/dirsearch)  | Alat yang digunakan untuk melakukan bruteforce terhadap direktori atau file sensitif yang terdapat pada website. |
|![John the Ripper](https://upload.wikimedia.org/wikipedia/commons/thumb/6/6d/John_the_ripper_pentesting_tool_logo.png/320px-John_the_ripper_pentesting_tool_logo.png)| [John the Ripper](https://www.openwall.com/john/) | Alat cracking password klasik yang mendukung deteksi otomatis jenis hash serta berbagai mode serangan (single crack, wordlist, incremental), sering digunakan untuk meng-crack file password hasil dump seperti `/etc/shadow`. |
|![Burp Suite Intruder](https://portswigger.net/content/images/logos/burp-suite-logo.svg)| [Burp Suite Intruder](https://portswigger.net/burp) | Modul pada Burp Suite yang memungkinkan brute force terhadap parameter HTTP request (login form, token, parameter API) dengan payload kustom serta analisis respons secara detail. |
|![Medusa](https://www.kali.org/tools/medusa/images/medusa-logo.svg)| [Medusa](https://www.kali.org/tools/medusa/) | Alat brute force login paralel yang mendukung banyak protokol jaringan, dirancang untuk kecepatan tinggi melalui proses multi-threading. |

## Penggunaan Tools

#### Hydra

Penggunaan Hydra dalam melakukan bruteforce suatu kredensial terhadap form login pada suatu website ataupun service jaringan seperti SSH atau FTP.

```
hydra -l admin -P /path/to/wordlist.txt localhost -s 4280 http-get "/vulnerabilities/brute/?username=^USER^&password=^PASS^&Login=Login"
```

Penjelasan command sebagai berikut:
- `-l`: spesifikasi username yang akan digunakan login, dalam kasus ini pada setiap percobaan login akan menggunakan username `admin`.
- `-P`: menggunakan password list yang dituju.
- `http-get`: agar Hydra mengetahui bahwa request yang dilakukan sebagai method GET.
- `"/vulnerabilities/brute/?username=^USER^&password=^PASS^&Login=Login"`:
    - `/vulnerabilities/brute/`: sebagai path yang akan dituju untuk bruteforce.
    - `^USER^`: placeholder username.
    - `^PASS^`: placeholder password.

![alt text](images/image.png)

#### Hashcat

Hashcat digunakan untuk meng-crack hash password yang telah didapatkan, misalnya dari hasil dump database atau file `/etc/shadow`.
 
Contoh penggunaan Hashcat dengan dictionary attack terhadap hash bertipe MD5:
 
```
hashcat -m 0 -a 0 hash.txt wordlist.txt
```
 
Penjelasan command:
- `-m 0`: menentukan mode hash yang digunakan, di mana `0` merepresentasikan algoritma MD5. Setiap algoritma hash (SHA1, NTLM, bcrypt, dsb) memiliki kode mode masing-masing yang dapat dilihat pada dokumentasi Hashcat.
- `-a 0`: menentukan jenis serangan yang digunakan, di mana `0` merepresentasikan straight/dictionary attack.
- `hash.txt`: file yang berisi hash yang ingin di-crack.
- `wordlist.txt`: wordlist yang digunakan sebagai kandidat password.
Untuk melakukan hybrid attack (kombinasi wordlist dengan mask tambahan angka), dapat menggunakan mode `-a 6`:


#### Dirsearch

Dirsearch digunakan untuk menemukan direktori maupun file tersembunyi pada sebuah website yang tidak terpublikasi secara langsung, seperti halaman admin, file backup, atau endpoint API yang tidak terdokumentasi.
 
Contoh penggunaan dasar Dirsearch:
 
```
dirsearch -u http://target.com -e php,html,js -w /path/to/wordlist.txt
```
 
Penjelasan command:
- `-u`: menentukan URL target yang akan di-scan.
- `-e`: menentukan ekstensi file yang ingin dicoba, dalam contoh ini `php`, `html`, dan `js`.
- `-w`: menentukan path wordlist yang berisi daftar nama direktori/file yang akan dicoba.

## Mitigasi

Beberapa langkah yang dapat diterapkan untuk mencegah atau memperlambat brute force attack:
 
- Account lockout: mengunci akun sementara setelah sejumlah percobaan login gagal berturut-turut.
- Rate limiting: membatasi jumlah request yang dapat dilakukan dari satu IP dalam rentang waktu tertentu.
- Menambahkan verifikasi Captcha: menambahkan verifikasi Captcha pada form login setelah beberapa kali percobaan gagal untuk mencegah otomasi.
- Multi-Factor Authentication atau MFA: menambahkan lapisan autentikasi kedua sehingga password saja tidak cukup untuk mengakses akun.
- Monitoring dan alerting: memantau log autentikasi untuk mendeteksi pola percobaan login yang mencurigakan secara dini.

## Referensi

1. CompTIA PenTest+ Student Guide (PT0-002)
2. https://wiki.owasp.org/index.php/Testing_for_Brute_Force_(OWASP-AT-004)
3. https://www.crowdstrike.com/cybersecurity-101/brute-force-attacks/
4. https://hashcat.net/wiki/
5. https://github.com/maurosoria/dirsearch
