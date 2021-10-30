# Jarkom-Modul-2-D07-2021
### Anggota kelompok:
Anggota | NRP | 
------------- | ------------- | 
Amanda Rozi Kurnia | 05111940000094 | 
Dyandra Paramitha W. | 05111940000119 |
Daanii Nabil Ghinannafsi Kusnanta | 05111940000163 |

## Daftar Soal
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

<image src="img/1a.PNG">

Kemudian pada setiap node ditambahkan `nameserver 192.168.122.1` ke dalam file di path `/etc/resolv.conf`, dengan menambahkannya pada `root/.bashrc` juga dengan command `echo "nameserver 192.168.122.1" > etc/resolv.conf` sehingga nameserver akan otomatis ditambahkan apabila node dinyalakan. Nameserver ini ditambahkan agar setiap node terhubung dengan router `Foosha` yang kita miliki.  

<image src="img/1b.PNG">


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

<image src="img/2a.PNG">

kemudian buat folder kaizoku di `/etc/bind` menggunakan `mkdir /etc/bind/kaizoku`. 

Lalu copy `/etc/bind/db.local` menjadi `/etc/bind/kaizoku/franky.d07.com` menggunakan `cp /etc/bind/db.local /etc/bind/kaizoku/franky.d07.com`. 

Lalu konfigurasi file tersebut seperti pada gambar. 

<image src="img/2b.PNG">

## <a name="soal3"></a> Soal 3
Setelah itu buat subdomain super.franky.yyy.com dengan alias www.super.franky.yyy.com yang diatur DNS nya di EniesLobby dan mengarah ke Skypie.

### Pembahasan
Pada file `/etc/bind/kaizoku/franky.d07.com` ditambahkan: 
```bash
        ns1     IN      A       192.195.2.4 ; IP Skypie
        super   IN      NS      ns1
```
<image src="img/3b.PNG">

Kemudian menambahkan pada `/etc/bind/named.conf.local`: 
```bash
zone "super.franky.d07.com" {
        type master;
        file "/etc/bind/kaizoku/super.franky.d07.com";
};
```
Untuk menambah subdomain `super.franky.d07.com`.


<image src="img/3a.PNG">

Kemudian buat file baru untuk subdomain seperti pada nomor di atas. Copy `/etc/bind/db.local` menjadi `/etc/bind/kaizoku/super.franky.d07.com` menggunakan `cp /etc/bind/db.local /etc/bind/kaizoku/super.franky.d07.com`. 

Lalu konfigurasi file tersebut seperti pada gambar. 
<image src="img/3c.PNG">

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

<image src="img/4a.PNG">

#### Loguetown
Pada server Loguetown, menjalankan `apt-get-update` dan `apt-get install dnsutils`. 

Selain itu juga menambahkan nameserver dari node Ennieslobby dan Skypie ke dalam file `/etc/resolv.conf`

<image src="img/4b.PNG">

#### Testing
```bash
host -t PTR 192.195.2.4
```
<image src="img/4c.PNG">

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

<image src="img/5a.PNG">

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

<image src="img/5c.PNG">

#### Pada Loguetown
Pada Loguetown ditambahkan nameserver Water7. 

#### Testing
- Mematikan service bind pada Ennieslobby `service bind9 stop`. 
- Melakukan `ping franky.d07.com`. 

<image src="img/5b.PNG">

## <a name="soal6"></a> Soal 6

Setelah itu terdapat subdomain mecha.franky.yyy.com dengan alias www.mecha.franky.yyy.com yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo.

### Pembahasan
#### Pada Ennieslobby
Mengedit file `/etc/bind/kaizoku/franky.d05.com`: 

```bash
        ns1     IN      A       192.195.2.3 ; IP Water7
        mecha   IN      NS      ns1
```

<image src="img/6a.PNG">


Mengedit file `/etc/bind/named.conf.options` dengan mengcomment bagian `dnssec-validation auto;` dan menambahkan `allow-query{any;};`. 

<image src="img/6b.PNG">

#### Pada Water7
Mengedit file `/etc/bind/named.conf.options` dengan mengcomment bagian `dnssec-validation auto;` dan menambahkan `allow-query{any;};`. 
<image src="img/6b.PNG">

Menambahkan zone pada `/etc/bind/named.conf.local` dengan menambahkan:

```bash
    zone "mecha.franky.d07.com" {
            type master;
            file "/etc/bind/sunnygo/mecha.franky.d07.com";
    };
```
<image src="img/6c.PNG">

Buat folder sunnygo dengan `mkdir /etc/bind/sunnygo` dan melakukan `cp /etc/bind/db.local /etc/bind/sunnygo/mecha.franky.d07.com`, dan melakukan konfigurasi pada gambar. 
<image src="img/6d.PNG">

#### Testing
Melakukan `ping mecha.franky.d07.com` pada server Loguetown. 
<image src="img/6e.PNG">

## <a name="soal7"></a> Soal 7

Untuk memperlancar komunikasi Luffy dan rekannya, dibuatkan subdomain melalui Franky dengan nama general.mecha.franky.yyy.com dengan alias www.general.mecha.franky.yyy.com yang mengarah ke Skypie.

### Pembahasan
#### Pada Water7: 
Mengedit file `/etc/bind/sunnygo/mecha.franky.d07.com`: 
<image src="img/7a.PNG">

Menambahkan zone pada `/etc/bind/named.conf.local`: 
```bash
    zone "general.mecha.franky.d07.com" {
            type master;
            file "/etc/bind/sunnygo/general.mecha.franky.d07.com";
    };
```
<image src="img/7b.PNG">

Melakukan `cp /etc/bind/db.local /etc/bind/sunnygo/general.mecha.franky.d07.com`, dan melakukan konfigurasi pada gambar.
<image src="img/7c.PNG"> 

#### Testing
Melakukan `ping general.mecha.franky.d07.com` pada server Loguetown. 
<image src="img/7d.PNG">

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
<image src="img/8b.PNG">

Kemudian membuat directory baru dengan nama `franky.d07.com` pada `/var/www/` (`mkdir /var/www/franky.d07.com`). lalu copy isi dari folder `franky` yang telah didownload ke `/var/www/franky.d07.com`. 
```bash
cp /root/Praktikum-Modul-2-Jarkom-main/franky/index.php /var/www/franky.d07.com
cp /root/Praktikum-Modul-2-Jarkom-main/franky/home.html /var/www/franky.d07.com
```

Setelah itu jalankan command `a2ensite franky.d07.com` dan `service apache2 restart`

#### Testing
Melakukan `lynx franky.d07.com` pada Loguetown. 

<image src="img/8c.PNG">



## <a name="kendala"></a> Kendala Yang Dialami



