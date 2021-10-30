# Jarkom-Modul-2-D07-2021
### Anggota kelompok:
Anggota | NRP | 
------------- | ------------- | 
Amanda Rozi Kurnia | 05111940000094 | 
Dyandra Paramitha W. | 05111940000119 |
Daanii Nabil Ghinannafsi Kusnanta | 05111940000163 |

## Notes
[Soal Shift 2](https://docs.google.com/document/d/11_xDG1yHMIOAZVPksRV0VDwx8S4KIgtAMCLh3UD5-qM/edit) <br>
**Prefix:** 192.195

## Daftar Isi
* [Soal 1](#soal1)
* [Soal 2](#soal2)
* [Soal 3](#soal3)
* [Soal 4](#soal4)
* [Soal 5](#soal5)
* [Soal 6](#soal6)
* [Soal 7](#soal7)
* [Soal 8](#soal8)
* [Soal 9](#soal9)
* [Soal 10](#soal10)
* [Soal 11](#soal11)
* [Soal 12](#soal12)
* [Soal 13](#soal13)
* [Soal 14](#soal14)
* [Soal 15](#soal15)
* [Soal 16](#soal16)
* [Soal 17](#soal17)
* [Kendala Yang Dialami](#kendala)

## <a name="soal1"></a> Soal 1
EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet

### Pembahasan:
Sebelumnya, telah dibuat topologi sesuai dengan yang diminta, dan pada Foosha dijalankan command `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.195.0.0/16` agar dapat terhubung ke jaringan luar pada router `Foosha`. Adapun agar `Foosha` dapat langsung terhubung dengan jaringan saat dijalankan, command disimpan dalam `root/.bashrc` yang automatis akan jalan apabila Foosha dinyalakan. 

<image src="img/1a.PNG" width="700">

Kemudian pada setiap node ditambahkan `nameserver 192.168.122.1` ke dalam file di path `/etc/resolv.conf`, dengan menambahkannya pada `root/.bashrc` juga dengan command `echo "nameserver 192.168.122.1" > etc/resolv.conf` sehingga nameserver akan otomatis ditambahkan apabila node dinyalakan. Nameserver ini ditambahkan agar setiap node terhubung dengan router `Foosha` yang kita miliki.  

<image src="img/1b.PNG" width="700">


## <a name="soal2"></a> Soal 2
Membuat website utama dengan mengakses franky.yyy.com dengan alias www.franky.yyy.com pada folder kaizoku

### Pembahasan
pada EniesLobby sebelumnya dijalankan command `apt-get update` dan `apt-get install bind9 -y` untuk menginstall bind9. 
Apabila bind9 sudah terinstall, edit file `named.conf.local` pada folder `bind` menggunakan command `vi /etc/bind/named.conf.local` dengan menambahkan: 

```bash
zone "franky.d07.com" {
	type master;
	file "/etc/bind/kaizoku/franky.d07.com";
};
```

<image src="img/2a.PNG" width="700">

kemudian buat folder kaizoku di `/etc/bind` menggunakan `mkdir /etc/bind/kaizoku`. 

Lalu copy `/etc/bind/db.local` menjadi `/etc/bind/kaizoku/franky.d07.com` menggunakan `cp /etc/bind/db.local /etc/bind/kaizoku/franky.d07.com`. 

Lalu konfigurasi file tersebut seperti pada gambar. 

<image src="img/2b.PNG" width="700">

## <a name="soal3"></a> Soal 3
Setelah itu buat subdomain super.franky.yyy.com dengan alias www.super.franky.yyy.com yang diatur DNS nya di EniesLobby dan mengarah ke Skypie.

### Pembahasan
Pada file `/etc/bind/kaizoku/franky.d07.com` ditambahkan: 
```bash
        ns1     IN      A       192.195.2.4 ; IP Skypie
        super   IN      NS      ns1
```
<image src="img/3b.PNG" width="700">

Kemudian menambahkan pada `/etc/bind/named.conf.local`: 
```bash
zone "super.franky.d07.com" {
        type master;
        file "/etc/bind/kaizoku/super.franky.d07.com";
};
```
Untuk menambah subdomain `super.franky.d07.com`.


<image src="img/3a.PNG" width="700">

Kemudian buat file baru untuk subdomain seperti pada nomor di atas. Copy `/etc/bind/db.local` menjadi `/etc/bind/kaizoku/super.franky.d07.com` menggunakan `cp /etc/bind/db.local /etc/bind/kaizoku/super.franky.d07.com`. 

Lalu konfigurasi file tersebut seperti pada gambar. 
<image src="img/3c.PNG" width="700">

## <a name="soal4"></a> Soal 4
Buat juga reverse domain untuk domain utama.

### Pembahasan

#### EnniesLobby
Mengedit file pada `etc/bind/named.conf.local`: 
```bash
zone "2.195.192.in-addr.arpa" {
	type master;
        file "/etc/bind/kaizoku/2.195.192.in-addr.arpa";
};
```
Copy file `/etc/bind/db.local` menjadi `/etc/bind/kaizoku/2.195.192.in-addr.arpa` dan melakukan konfigurasi seperti pada gambar. 

<image src="img/4a.PNG" width="700">

#### Loguetown
Pada server Loguetown, menjalankan `apt-get-update` dan `apt-get install dnsutils`. 

Selain itu juga menambahkan nameserver dari node Ennieslobby dan Skypie ke dalam file `/etc/resolv.conf`

<image src="img/4b.PNG" width="700">

#### Testing
```bash
host -t PTR 192.195.2.4
```
<image src="img/4c.PNG" width="700">

## <a name="soal5"></a> Soal 5
Supaya tetap bisa menghubungi Franky jika server EniesLobby rusak, maka buat Water7 sebagai DNS Slave untuk domain utama. 

### Pembahasan
#### Pada Ennieslobby
Mengedit file pada `etc/bind/named.conf.local` dan melakukan replace pada zone `franky.d07.com`: 
```bash
zone "franky.d07.com" {
            type master;
            notify yes;
            also-notify { 192.195.2.3; };
            allow-transfer { 192.195.2.3; };
            file "/etc/bind/kaizoku/franky.d07.com";
};

```
Kemudian melakukan restart pada bind. `service bind9 restart`. 

<image src="img/5a.PNG" width="700">

#### Pada Water7
Sama seperti pada server Ennieslobby, jalankan command `apt-get update` dan `apt-get install bind9 -y` untuk menginstall bind9. 

Kemudian pada `/etc/bind/named.conf.local` ditambahkan:
```bash
zone "franky.d07.com" {
    type slave;
    masters { 192.195.2.2; };
    file "/var/lib/bind/franky.d07.com";
};
```
Kemudian melakukan restart pada bind. `service bind9 restart`. 

<image src="img/5c.PNG" width="700">

#### Pada Loguetown
Pada Loguetown ditambahkan nameserver Water7. 

#### Testing
- Mematikan service bind pada Ennieslobby `service bind9 stop`. 
- Melakukan `ping franky.d07.com`. 

<image src="img/5b.PNG" width="700">

## <a name="soal6"></a> Soal 6

Setelah itu terdapat subdomain mecha.franky.yyy.com dengan alias www.mecha.franky.yyy.com yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo.

### Pembahasan
#### Pada Ennieslobby
Mengedit file `/etc/bind/kaizoku/franky.d05.com`: 

```bash
        ns1     IN      A       192.195.2.3 ; IP Water7
        mecha   IN      NS      ns1
```

<image src="img/6a.PNG" width="700">


Mengedit file `/etc/bind/named.conf.options` dengan mengcomment bagian `dnssec-validation auto;` dan menambahkan `allow-query{any;};`. 

<image src="img/6b.PNG" width="700">

#### Pada Water7
Mengedit file `/etc/bind/named.conf.options` dengan mengcomment bagian `dnssec-validation auto;` dan menambahkan `allow-query{any;};`. <br>
<image src="img/6b.PNG" width="700">

Menambahkan zone pada `/etc/bind/named.conf.local` dengan menambahkan:

```bash
    zone "mecha.franky.d07.com" {
            type master;
            file "/etc/bind/sunnygo/mecha.franky.d07.com";
    };
```
<image src="img/6c.PNG" width="700">

Buat folder sunnygo dengan `mkdir /etc/bind/sunnygo` dan melakukan `cp /etc/bind/db.local /etc/bind/sunnygo/mecha.franky.d07.com`, dan melakukan konfigurasi pada gambar. <br>
<image src="img/6d.PNG" width="700">

#### Testing
Melakukan `ping mecha.franky.d07.com` pada server Loguetown. 
<image src="img/6e.PNG" width="700">

## <a name="soal7"></a> Soal 7

Untuk memperlancar komunikasi Luffy dan rekannya, dibuatkan subdomain melalui Franky dengan nama general.mecha.franky.yyy.com dengan alias www.general.mecha.franky.yyy.com yang mengarah ke Skypie.

### Pembahasan
#### Pada Water7: 
Mengedit file `/etc/bind/sunnygo/mecha.franky.d07.com`: 
<image src="img/7a.PNG" width="700">

Menambahkan zone pada `/etc/bind/named.conf.local`: 
```bash
    zone "general.mecha.franky.d07.com" {
            type master;
            file "/etc/bind/sunnygo/general.mecha.franky.d07.com";
    };
```
<image src="img/7b.PNG" width="700">

Melakukan `cp /etc/bind/db.local /etc/bind/sunnygo/general.mecha.franky.d07.com`, dan melakukan konfigurasi pada gambar.
<image src="img/7c.PNG" width="700"> 

#### Testing
Melakukan `ping general.mecha.franky.d07.com` pada server Loguetown. 
<image src="img/7d.PNG" width="700">

## <a name="soal8"></a> Soal 8
Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.franky.yyy.com. Pertama, luffy membutuhkan webserver dengan DocumentRoot pada /var/www/franky.yyy.com.

### Pembahasan
### Pada Loguetown
Melakukan instalasi `lynx` menggunakan `apt-get update` dan `apt-get install lynx`

#### Pada Skypie
Melakukan penginstallan `apache2`, `php`, dan `lbapache2-mod-php7.0`.

```bash
apt-get install apache2
service apache2 start
apt-get install php
apt-get install libapache2-mod-php7.0
service apache2 restart
```

Jangan lupa untuk melakukan `wget` pada file github pada modul dan melakukan unzip. 

*Apbila tidak bisa, install terlebih dahulu wget dan unzipnya. 

```bash
wget https://github.com/FeinardSlim/Praktikum-Modul-2-Jarkom/archive/refs/heads/main.zip
```

Kemudian pindah ke directory `/etc/apache2/sites-available` (`cd /etc/apache2/sites-available`) dan melakukan copy file `cp 000-default.conf franky.d07.com.conf`. 

Lalu setting file `franky.d07.com.conf` seperti pada gambar:
<image src="img/8b.PNG" width="700">

Kemudian membuat directory baru dengan nama `franky.d07.com` pada `/var/www/` (`mkdir /var/www/franky.d07.com`). lalu copy isi dari folder `franky` yang telah didownload ke `/var/www/franky.d07.com`. 
```bash
cp /root/Praktikum-Modul-2-Jarkom-main/franky/index.php /var/www/franky.d07.com
cp /root/Praktikum-Modul-2-Jarkom-main/franky/home.html /var/www/franky.d07.com
```

Setelah itu jalankan command `a2ensite franky.d07.com` dan `service apache2 restart`

#### Testing
Melakukan `lynx franky.d07.com` pada Loguetown. 

<image src="img/8c.PNG" width="700">

## <a name="soal9"></a> Soal 9
Setelah itu, Luffy juga membutuhkan agar url www.franky.yyy.com/index.php/home dapat menjadi menjadi www.franky.yyy.com/home. 

### Pembahasan
### Pada Skypie
Pertama, aktifkan modul `rewrite` dengan menjalankan perintah `a2enmod rewrite`. Kemudian, restart apache2 `service apache2 restart` dan pindah ke directory `var/www/franky.d07.com`. Buat file `.htaccess` dengan isi
```bash
	RewriteEngine On
	RewriteRule ^home$ index.php/home
```
<image src="img/9a.PNG" width="700">
	
Selanjutnya, buka file ```/etc/apache2/sites-available/franky.d07.com.conf``` dan tambahkan
```bash
	<Directory /var/www/franky.d07.com>
		Options +FollowSymLinks -Multiviews
		AllowOverride All
	</Directory>
```
	
<!-- image -->

### Pada Loguetown
Lakukan testing pada client dengan menjalankan command `lynx franky.d07.com/home`. Akan muncul halaman:
<image src="img/9c.PNG" width="700">
	
## <a name="soal10"></a> Soal 10
Setelah itu, pada subdomain www.super.franky.yyy.com, Luffy membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/super.franky.yyy.com

### Pembahasan
Pindah ke directory `/etc/apache2/sites-available`, kemudian copy file `000-default.conf` menjadi file `super.franky.d07.com.conf`.
```bash
	cd /etc/apache2/sites-available
	cp 000-default.conf super.franky.d07.com.conf
```

<!-- image 10a -->

Lalu, lakukan setting pada file `super.franky.d07.com.conf` dengan line berikut:
```bash
	ServerAdmin webmaster@localhost
        #DocumentRoot /var/www/html
        ServerName super.franky.d07.com
        ServerAlias www.super.franky.d07.com
        DocumentRoot /var/www/super.franky.d07.com
```

<!-- image 10b -->

Buat directory baru dengan nama `super.franky.d07.com` pada directory `/var/www` menggunakan `mkdir /var/www/super.franky.d07.com`. Selanjutnya, copy isi folder `super.franky` yang telah didownload ke `/var/www/super.franky.d07.com`.
```bash
	cp -r /root/Praktikum-Modul-2-Jarkom-main/super.franky/error /var/www/super.franky.d07.com
	cp -r /root/Praktikum-Modul-2-Jarkom-main/super.franky/public /var/www/super.franky.d07.com
```

Jalankan command `a2ensite super.franky.d07.com` dan `service apache2 restart`.

<!-- image 10c -->

### Pada Loguetown
Lakukan testing pada client dengan command `lynx super.franky.d07.com`. Maka, akan muncul halaman berikut.

<!-- image 10d -->
	
## <a name="soal11"></a> Soal 11
Akan tetapi, pada folder /public, Luffy ingin hanya dapat melakukan directory listing saja.
	
### Pembahasan
Pindah ke directory `/etc/apache2/sites-available` kemudian buka file `super.franky.d07.com.conf` dan tambahkan:
```bash
	<Directory /var/www/super.franky.d07.com/public>
	    Options +Indexes
	</Directory>
```

<!-- image 11a -->

Jalankan command `service apache2 restart`.

### Pada Loguetown
Lakukan testing pada client dengan menjalankan command `lynx super.franky.d07.com/public`. Maka, akan muncul halaman sebagai berikut:

<!-- image 11b -->
	
## <a name="soal12"></a> Soal 12
Tidak hanya itu, Luffy juga menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache
	
### Pembahasan
Pindah ke directory `/etc/apache2/sites-available` kemudian buka file `vi super.franky.d07.com.conf` dan tambahkan:
```bash
	ErrorDocument 404 /error/404.html
```

<!-- image 12a -->

### Pada Loguetown
Lakukan testing pada client dengan menjalankan command `lynx super.franky.d07.com/halo` (lokasi yang tidak ada). Maka akan muncul halaman berikut:

<!-- image 12b -->
<image src="img/12b.PNG" width="700">
	
## <a name="soal13"></a> Soal 13
Luffy juga meminta Nami untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.super.franky.yyy.com/public/js menjadi www.super.franky.yyy.com/js. 

### Pembahasan
Buka file `super.franky.d07.com.conf` kemudian tambahkan isinya dengan
```bash
	Alias "/js" "/var/www/super.franky.d07.com/public/js"
```

<!-- image 13 -->
	
## <a name="soal14"></a> Soal 14
Dan Luffy meminta untuk web www.general.mecha.franky.yyy.com hanya bisa diakses dengan port 15000 dan port 15500
	
### Pembahasan
### Pada Skypie
Pindah ke `/etc/apache2/sites-available` lalu copy file `000-default.conf` menjadi file `general.mecha.franky.d07.com.conf`.
```bash
	cp 000-default.conf general.mecha.franky.d07.com.conf
	vi general.mecha.franky.d07.com
```

Setelah itu, buka file tersebut dan tambahkan isinya sebagai berikut.
```bash
	<VirtualHost *:15000 *:15500>
        # The ServerName directive sets the request scheme, hostname and port thh
	at
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        #DocumentRoot /var/www/html
        ServerName general.mecha.franky.d07.com
        ServerAlias www.general.mecha.franky.d07.com
        DocumentRoot /var/www/general.mecha.franky.d07.com
```

Kemudian, buka `etc/apache2/ports.conf` dan tambahkan
```bash
	Listen 15000
	Listen 15500
```

Buat directory baru dengan nama `general.mecha.franky.d07.com` pada `/var/www/`. Setelah itu, copy isi dari folder `general.mecha.franky` yang telah didownload ke `/var/www/general.mecha.franky.d07.com`.
```bash
	mkdir /var/www/general.mecha.franky.d07.com
	cp /root/Praktikum-Modul-2-Jarkom-main/general.mecha.franky/* /var/www/general.mecha.franky.d07.com
```

Kemudian, jalankan perintah `a2ensite general.mecha.franky.d07.com` dan `service apache2 restart`.

## <a name="soal15"></a> Soal 15
Dengan autentikasi username luffy dan password onepiece dan file di /var/www/general.mecha.franky.yyy
	
### Pembahasan
### Pada Skypie
Jalankan perintah `htpasswd -c /etc/apache2/.htpasswd luffy` untuk membuat file yang menyimpan username dan password ke dalam file `/etc/apache2/.htpasswd` dengan user `luffy`. Masukkan password: `onepiece`.
	
Kemudian, buka file `/etc/apache2/sites-available/general.mecha.franky.d07.com.conf` dan edit isinya menjadi:
```bash
	<Directory /var/www/general.mecha.franky.d07.com>
		Options +FollowSymLinks -Multiviews
		AllowOverride All
	</Directory>
```

Setelah itu, buka file `/var/www/general.mecha/franky.d07.com/.htaccess` dan tambahkan isinya dengan:
```bash
	AuthType Basic
	AuthName "Restricted Content"
	AuthUserFile /etc/apache2/.htpasswd
	Require valid-user
```
Kemudian, jalankan perintah `service apache2 restart`.

### Pada Loguetown
Lakukan testing pada client dengan menjalankan perintah `lynx http://192.195.2.4:15000` dan `lynx http://192.195.2.4:15500`.
		
## <a name="soal16"></a> Soal 16
Dan setiap kali mengakses IP Skypie akan dialihkan secara otomatis ke www.franky.yyy.com
	
### Pembahasan
### Pada Skypie
Pindah ke directory `/var/www/html` dan edit file `.htaccess` menjadi:
```bash
	RewriteEngine On
	RewriteBase /~new/
	RewriteCond %{HTTP_HOST} ^192\.195\.2\.4$
	RewriteRule ^(.*)$ franky.d07.com [L,R=301]
```

Lalu, buka file `/etc/apache2/sites-available/000-default.conf` dan edit isinya menjadi:
```bash
	<Directory /var/www/html>
		Options +FollowSymLinks -Multiviews
		AllowOverride All
	</Directory>
```

Kemudian, jalankan perintah `service apache2 restart`.

### Pada Loguetown
Lakukan testing pada client dengan menjalankan perintah `lynx http://192.195.2.4`.

## <a name="soal17"></a> Soal 17
Dikarenakan Franky juga ingin mengajak temannya untuk dapat menghubunginya melalui website www.super.franky.yyy.com, dan dikarenakan pengunjung web server pasti akan bingung dengan randomnya images yang ada, maka Franky juga meminta untuk mengganti request gambar yang memiliki substring “franky” akan diarahkan menuju franky.png.
	
### Pembahasan
### Pada Skypie
Gunakan Rewrite module untuk redirect semua request access yang mengandung kata 'franky' ke file `/public/images/franky.png`. Buka file `/etc/apache2/sites-available/super.franky.d07.com.conf` dan tambahkan command berikut:
```bash
	<Directory /var/www/super.franky.d07.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
```

Kemudian, buka `/var/www/super.franky.d07.com/.htaccess` dan tambahkan command berikut:
```bash
	RewriteEngine ON
	RewriteRule ^(.*)franky(.*)$ http://super.franky.d07.com/public/images/franky.pnn
	g [L,R]
```

### Pada Loguetown
Lakukan testing pada client dengan menjalankan `lynx www.super.franky.d07.com/public/images/franky.jpg.`

## <a name="kendala"></a> Kendala Yang Dialami
* Node selain Foosha tidak bisa terhubung ke internet
* Penulisan syntax yang typo sehingga sulit melakukan debugging

## <a name="referensi"></a> Referensi
* https://www.bluehost.com/help/article/url-redirect-rewrite-using-the-htaccess-file


