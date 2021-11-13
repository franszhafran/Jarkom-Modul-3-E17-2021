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
