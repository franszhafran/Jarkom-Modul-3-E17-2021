# Modul 3

## 1. Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server
EniesLobby telah diatur sebagai DNS Server pada Modul 2. Install DHCP server pada Jipangu dan Squid pada Water7.

- Jipangu
File config `/etc/default/isc-dhcp-server` berisi:
```
INTERFACES="eth0"
```

- Water7
File config `/etc/squid/squid.conf` berisi:
```
http_port 5000
visible_hostname Water7
```
agar aktif sebagai proxy server

## 2. Foosha sebagai DHCP Relay
Install isc-dhcp-relay di Foosha dan edit file config `/etc/default/isc-dhcp-relay` berisi:
```
# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts

#
# This is a POSIX shell fragment
#

# What servers should the DHCP relay forward requests to?
SERVERS="192.208.2.4"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2 eth3"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```
Agar meneruskan DHCP Request dari eth1, eth2, eth3 ke DHCP Server


## 3. Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169.
Agar client menggunakan config ip dari DHCP server, maka pengaturan network configuration nya adalah seperti ini
```
auto eth0
iface eth0 inet dhcp
```

Lalu pada DHCP Server, file config `/etc/dhcp/dhcpd.conf` atur sebagai berikut:
```
subnet 192.208.1.0 netmask 255.255.255.0 {
    range 192.208.1.20 192.208.1.99;
    range 192.208.1.150 192.208.1.169;
    option routers 192.208.1.1;
    option broadcast-address 192.208.1.255;
    option domain-name-servers 192.208.2.2;
    default-lease-time 360;
    max-lease-time 7200;
}
```
Agar subnet mendapatkan ip dengan range yang diinginkan

## 4. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50 
Untuk subnet Switch3, atur sebagai berikut agar mendapatkan ip range yang diinginkan.
```
subnet 192.208.3.0 netmask 255.255.255.0 {
    range 192.208.3.30 192.208.3.50;
    option routers 192.208.3.1;
    option broadcast-address 192.208.3.255;
    option domain-name-servers 192.208.2.2;
    default-lease-time 720;
    max-lease-time 7200;
}
```

## 5. Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.
Untuk pengaturan dns, terdapat pengaturan dari dhcp server pada baris
```
option domain-name-servers 192.208.2.2;
```
sehingga client akan menggunakan alamat dns tersebut.

Lalu pada DNS Server, atur file `/etc/bind/named.conf.options` sebagai berikut:
```
options {
        forwarders {
                192.168.122.1;
                1.1.1.1;
        };
        directory "/var/cache/bind";

        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```
Dengan seperti ini, pertanyaan domain yang tidak bisa dijawab EniesLobby akan diteruskan ke DNS luar yang tersambung dengan internet.

## 6. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit. 
- Switch1
Pengaturan lease time pada nomor 4, tepatnya pada baris:
```
default-lease-time 360;
max-lease-time 7200;
```
mengatur waktu peminjaman 6 menit dan maksimal ip 120 menit

- Switch3
- Switch1

Pengaturan lease time pada nomor 4, tepatnya pada baris:
```
default-lease-time 720;
max-lease-time 7200;
```
mengatur waktu peminjaman 12 menit dan maksimal ip 120 menit

## 7. Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69
Edit network config pada Skypie dan atur MAC Address nya:
```
auto eth0
iface eth0 inet dhcp
hwaddress ether 52:21:08:91:8d:49
```

Pada Jipangu (DHCP Server) buat config agar MAC Address Skypie mendapatkan IP 192.208.3.69
```
host Skypie {
    hardware ethernet 52:21:08:91:8d:49;
    fixed-address 192.208.3.69;
}
```

## 8. Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi. Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000
Tambahkan domain jualbelikapal.e17.com ke DNS Server agar dikenali dengan tujuan IP ke mesin Proxy Server
Pengaturan `/etc/bind/named.conf.local`
```zone "e17.com" {
    type master;
    file "/etc/bind/kaizoku/e17.com";
};
```

Pengaturan SOA untuk domain e17.com:
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     e17.com. root.e17.com. (
                        2021102401      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      e17.com.
@       IN      A       192.208.2.4 ;IP EniesLobby
www     IN      CNAME   e17.com.
jualbelikapal   IN      A       192.208.2.3 ;IP Skypie
www.jualbelikapal       IN      CNAME   super
```
jualbelikapal diatur sebagai cname dari root domain e17.com

Lalu pada Proxy Server di file `/etc/squid/squid.conf` atur sebagai berikut agar menggunakan port 5000:
```
http_port 5000
visible_hostname Water7
```

## 9. Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy 
Buat dahulu file credentialsnya dengan perintah `htpasswd -cm /etc/squid/passwd luffybelikapale17` untuk membuat file credential, kemudian tambahkan untuk user zoro dengan perintah `htpasswd -m /etc/squid/passwd zorobelikapale17`. Option -c untuk pembuatan pertama, option -m agar menggunakan enkripsi md5.

Kemudian atur file `/etc/squid/squid.conf` agar proxy menggunakan authentikasi:
```
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
```

## 10. Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)
Buat ACL untuk mengontrol jam yang diperbolehkan
```
acl SESI_SATU time MTWH 07:00-11:00
acl SESI_DUA time TWHF 17:00-24:00
acl SESI_TIGA time WHFA 00:00-03:00
```

Kemudian tambahkan `http_access allow` untuk ACL yang diinginkan (user yang terautentikasi dan pada sesi jam yang benar):
```
http_access allow SESI_SATU USERS
http_access allow SESI_DUA USERS
http_access allow SESI_TIGA USERS
```
Kemudian, tambahkan http_access allow untuk ACL tersebut, gabungkan dengan ACL 

## 11. Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie
Untuk memenuhi keinginan diatas, buat ACL yang akan mendeteksi bila domain tujuan google.com, lalu deny http_access yang masuk ke ACL tersebut. Kemudian atur deny_info untuk ACL yang tadi agar mengarah ke super.franky.e17.com
```
acl GOOGLE dstdomain .google.com
http_access deny GOOGLE
deny_info 301:http://super.franky.e17.com%R GOOGLE
```

## 12. Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps 
Untuk memenuhi keinginan, buat ACL untuk request ke gambar (png/jpg). Buat juga ACL untuk request dari user proxy luffy. Kemudian atur delay pool, class, access parameters dengan delay_parameteres 1250/1250 karena keinginan di soal adalah 10 kb sedangkan Squid membaca dengan bytes (B), maka 10kb perlu kita ubah menjadi byte (B) menjadi 1250 kB. 

```
acl IMAGES url_regex png$|jpg$
acl SILUFFY proxy_auth_regex luffy
delay_pools 1
delay_class 1 1
delay_access 1 allow SILUFFY IMAGES
delay_parameters 1 1250/1250
```

## 13. Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya 
Tidak ada pengaturan apapun karena yang dibatasi hanya user luffy
