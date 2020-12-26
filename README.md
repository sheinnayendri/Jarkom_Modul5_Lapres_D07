# Jarkom_Modul5_Lapres_D07
Laporan Resmi Modul 5 Praktikum Jaringan Komputer 2020
#
1. Naufal Adam Kuncoro (05111740000155)
2. Sheinna Yendri (05111840000038)
#

## A. Topologi Jaringan

![image](https://user-images.githubusercontent.com/48936125/102958243-0ecbdf00-450f-11eb-8571-442615416370.png)

Berdasarkan gambar topologi di atas, dibuatlah file **topologi.sh** sebagai berikut:
```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &
uml_switch -unix switch4 > /dev/null < /dev/null &
uml_switch -unix switch5 > /dev/null < /dev/null &
uml_switch -unix switch6 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.78.33 eth1=daemon,,,switch5 eth2=daemon,,,switch3 mem=96M &
xterm -T KEDIRI -e linux ubd0=KEDIRI,jarkom umid=KEDIRI eth0=daemon,,,switch1 eth1=daemon,,,switch6 eth2=daemon,,,switch5  mem=96M &
xterm -T BATU -e linux ubd0=BATU,jarkom umid=BATU eth0=daemon,,,switch3 eth1=daemon,,,switch4 eth2=daemon,,,switch2  mem=96M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=128M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch1 mem=128M &
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch1 mem=128M &

# Klien
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch6 mem=96M &
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch4 mem=96M &
```

## B. Subnetting CIDR

![S__19079595](https://user-images.githubusercontent.com/48936125/102977968-5580ff80-4536-11eb-9831-5eaef1afb080.jpg)

![S__19079597](https://user-images.githubusercontent.com/48936125/102977965-531ea580-4536-11eb-9e0c-d43497388cc4.jpg)

Dilakukan subnetting seperti pada gambar di atas, di mana untuk subnet Malang-Mojokerto-Batu tidak perlu diikutkan dalam pembagian IP karena mereka langsung mendapatkan **IP DMZ: 10.151.79.64/29**.

Untuk router yaitu **SURABAYA**, **BATU**, dan **KEDIRI** harus di-uncomment pada perintah ```net.ipv4.ip_forward=1``` agar dapat meneruskan route nantinya. Hal ini dilakukan dengan cara mengetikkan ```nano /etc/sysctl.conf``` kemudian edit di situ, dan untuk mengaktifkan perubahan baru, mengetikkan ```sysctl -p```.

Kemudian karena setiap subnet sudah mendapatkan pembagian IP, perlu dilakukan setting pada file ```/etc/network/interfaces``` pada setiap UML masing-masing sebagai berikut:

**SURABAYA (Sebagai Router)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.78.34
netmask 255.255.255.252
gateway 10.151.78.33

auto eth1
iface eth1 inet static
address 192.168.2.1
netmask 255.255.255.252

auto eth2
iface eth2 inet static
address 192.168.5.1
netmask 255.255.255.252
```

**KEDIRI (Sebagai Router dan DHCP Relay)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.2.1
netmask 255.255.255.248

auto eth1
iface eth1 inet static
address 192.168.0.1
netmask 255.255.255.0

auto eth2
iface eth2 inet static
address 192.168.1.2
netmask 255.255.255.252
gateway 192.168.1.1
```

**BATU (Sebagai Router dan DHCP Relay)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.5.2
netmask 255.255.255.252
gateway 192.168.5.1

auto eth1
iface eth1 inet static
address 192.168.4.1
netmask 255.255.255.0

auto eth2
iface eth2 inet static
address 10.151.79.65
netmask 255.255.255.248
```

**MALANG (Sebagai DNS Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.79.66
netmask 255.255.255.248
gateway 10.151.79.65
```

**MOJOKERTO (Sebagai DHCP Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.79.67
netmask 255.255.255.248
gateway 10.151.79.65
```

**MADIUN (Sebagai Web Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.1.2
netmask 255.255.255.248
gateway 192.168.1.1
```

**PROBOLINGGO (Sebagai Web Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.1.3
netmask 255.255.255.248
gateway 192.168.1.1
```

Untuk UML **GRESIK** dan **SIDOARJO** akan dibahas pada **subbab D** nantinya, karena mendapatkan pembagian IP bukan secara static, melainkan melalui DHCP.

Sedangkan untuk UML yang file ```/etc/network/interfaces``` sudah diedit, akan direstart networking-nya dengan perintah ```service networking restart```.

## C. Routing
Berdasarkan topologi pada soal A, karena menggunakan teknik subnetting CIDR, maka routing yang dibutuhkan lebih sederhana, hanya perlu dilakukan routing di UML **SURABAYA** sebagai berikut:
```
route add -net 10.151.79.64 netmask 255.255.255.248 gw 192.168.5.2
route add -net 192.168.4.0 netmask 255.255.254.0 gw 192.168.5.2
route add -net 192.168.0.0 netmask 255.255.252.0 gw 192.168.2.2
```

Agar routing tidak perlu dilakukan berulang-ulang, disimpan pada file bash misalkan dengan nama **route.sh** kemudian untuk menjalankannya dengan perintah ```source route.sh```.

## D. DHCP Server-Relay (IP GRESIK & SIDOARJO)
Karena  **MOJOKERTO** mnejadi DHCP Server, perlu diinstallkan dengan perintah ```apt-get install isc-dhcp-server```, sedangkan **KEDIRI** dan **BATU** yang akan menjadi DHCP Relay, perlu diinstallkan dengan perintah ```apt-get install isc-dhcp-relay```.

Sedangkan untuk klien **GRESIK** dan **SIDOARJO** akan diedit file ```/etc/network/interfaces``` menjadi sebagai berikut:
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
```

Kemudian pada DHCP Server (**MOJOKERTO**), diedit pada file ```/etc/default/isc-dhcp-server```, ditambahkan interface ```eth0``` untuk INTERFACESv4.

Kemudian diedit juga pada file ```/etc/dhcp/dhcpd.conf``` sebagai berikut:
```
subnet 10.151.79.65 netmask 255.255.255.248 {
}

subnet 192.168.4.0 netmask 255.255.255.0 {
	range 192.168.4.2 192.168.4.254;
	option routers 192.168.4.1;
	option broadcast-address 192.168.4.255;
	option domain-name-servers 10.151.79.66;
	default-lease-time 600;
	max-lease-time 600;
}

subnet 192.168.0.0 netmask 255.255.255.0 {
	range 192.168.0.2 192.168.0.254;
	option routers 192.168.0.1;
	option broadcast-address 192.168.0.255;
	option domain-name-servers 10.151.79.66;
	default-lease-time 600;
	max-lease-time 600;
}
```

Kemudian DHCP dapat direstart dengan perintah ```service isc-dhcp-server restart```.

Sedangkan untuk kedua DHCP Relay yaitu **KEDIRI** dan **BATU** akan diset untuk mendengarkan DHCP Server diberikan IP **MOJOKERTO** yaitu 10.151.79.67.

Sehingga ketika pada klien **GRESIK** dan **SIDOARJO** dilakukan ```service networking restart```, mereka akan mendapatkan IP secara dinamis sesuai range subnet mereka masing-masing yang diberikan oleh DHCP Server (**MOJOKERTO**). Dapat dicek apakah IP sudah berhasil dengan ```ifconfig```.

## 1. SURABAYA bisa akses keluar tanpa MASQUERADE
Ditambahkan perintah iptables sebagai berikut di **SURABAYA**:
```
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j SNAT --to-source 10.151.78.34
```

## 2. Akses SSH di luar topologi (UML) akan di-DROP ketika mengakses server yang memiliki IP DMZ (lakukan setting di SURABAYA)
Ditambahkan perintah iptables sebagai berikut di **SURABAYA**:
```
iptables -N LOGGING
iptables -A FORWARD -p tcp --dport 22 -d 10.151.79.64/29 -i eth0 -j LOGGING
iptables -A LOGGING -m limit --limit 5/min -j LOG --log-prefix "iptables_FORWARD_denied: " --log-level 7
iptables -A LOGGING -j DROP
```
Diberikan beberapa syntax tambahan berupa logging untuk memenuhi soal no.7 yaitu melakukan logging untuk semua paket yang di-DROP. SSH memiliki nomor port 22.

## 3. DHCP Server dan DNS Server maksimal menerima 3 koneksi ICMP secara bersamaan, selebihnya di-DROP (disetting pada masing-masing server)
Ditambahkan perintah iptables sebagai berikut di **MALANG (DNS Server)** dan **MOJOKERTO (DHCP Server)**:
```
iptables -N LOGGING
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j LOGGING
iptables -A LOGGING -m limit --limit 5/min -j LOG --log-prefix "iptables_FORWARD_denied: " --log-level 7
iptables -A LOGGING -j DROP
```
Diberikan beberapa syntax tambahan berupa logging untuk memenuhi soal no.7 yaitu melakukan logging untuk semua paket yang di-DROP.

## 4-5. SIDOARJO dan GRESIK diberikan waktu akses untuk mengakses server MALANG:
**SIDOARJO = 07:00 - 17:00 (Senin - Jumat)**
**GRESIK = 17:00 - 07:00 (Setiap Hari)**
Ditambahkan perintah iptables sebagai berikut di **MALANG**:
```
iptables -A INPUT -s 192.168.4.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 17:00 --timestop 23:59 -j ACCEPT
iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 00:00 --timestop 07:00 -j ACCEPT
iptables -A INPUT -s 192.168.0.0/24 -j REJECT
iptables -A INPUT -s 192.168.4.0/24 -j REJECT
```

## 6. Ketika mengakses DNS Server akan secara bergantian didistribusikan ke PROBOLINGGO dan MADIUN pada port 80 (setting iptables di SURABAYA)
Ditambahkan perintah iptables sebagai berikut di **SURABAYA**:
```
iptables -A PREROUTING -t nat -p tcp -d 192.168.6.1 --dport 80 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.1.2
iptables -t nat -A PREROUTING -p tcp -d 192.168.6.1 --dport 80 -j DNAT --to-destination 192.168.1.3
iptables -t nat -A POSTROUTING -p tcp -d 192.168.1.2 --dport 80 -j SNAT --to-source 192.168.6.1
iptables -t nat -A POSTROUTING -p tcp -d 192.168.1.3 --dport 80 -j SNAT --to-source 192.168.6.1
```

Pada **PROBOLINGGO** dan **MADIUN** yang merupakan Web Server harus dilakukan instalasi apache2 dengan perintah ```apt-get install apache2```.

Kemudian pada DNS Server yaitu **MALANG** akan diinstall bind9 dengan perintah ```apt-get install bind9```. Setelah itu dilakukan edit file ```/etc/bind/named.conf.local``` dan tambahkan domain baru misalkan **jarkomd07.com** sebagai berikut:
```
zone "jarkomd07.com" {
    type master;
    file "/etc/bind/jarkom/jarkomd07.com";
};
```

Kemudian buat folder baru: ```mkdir /etc/bind/jarkom```

Dan copy file **db.local** ke folder yang baru saja dibuat dan mengganti namanya sesuai domain yang diinginkan: ```cp /etc/bind/db.local /etc/bind/jarkom/jarkomd07.com```
Kemudian buka file semerud07.pw dengan perintah: nano /etc/bind/jarkom/semerud07.pw.

Edit pointer **A** menjadi ke IP yang belum digunakan yaitu **192.168.6.1** serta ganti localhost menjadi nama domain yaitu **jarkomd07.com**.

Setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

Untuk mengecek, misalkan pada klien **SIDOARJO** dijalankan ```nc jarkomd07.com 80``` kemudian enter dan ketik apapun misalkan ```tes```. Kemudian pada **MADIUN** dan **PROBOLINGGO** akan dicoba lihat apakah diterima dan di mana diterimanya dengan perintah ```tail /var/log/apache2/access.log```. Kemudian lakukan lagi, dan sekarang harusnya diterima di UML yang lainnya (bergantian), berarti berhasil.

## 7. Logging semua paket yang di-DROP
Untuk soal ini konfigurasi akan digabung bersama perintah iptables DROP yaitu untuk **nomor 2 dan 3** di atas.

Karena sistem UML semua iptables akan hilang tiap direstart, sebaiknya disimpan dalam file bash script misalkan **iptables.sh** pada tiap UML yang disetting sebagai berikut:

**SURABAYA**
```
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j SNAT --to-source 10.151.78.34
iptables -N LOGGING
iptables -A FORWARD -p tcp --dport 22 -d 10.151.79.64/29 -i eth0 -j LOGGING
iptables -A LOGGING -m limit --limit 5/min -j LOG --log-prefix "iptables_FORWARD_denied: " --log-level 7
iptables -A LOGGING -j DROP
iptables -A PREROUTING -t nat -p tcp -d 192.168.6.1 --dport 80 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.1.2
iptables -t nat -A PREROUTING -p tcp -d 192.168.6.1 --dport 80 -j DNAT --to-destination 192.168.1.3
iptables -t nat -A POSTROUTING -p tcp -d 192.168.1.2 --dport 80 -j SNAT --to-source 192.168.6.1
iptables -t nat -A POSTROUTING -p tcp -d 192.168.1.3 --dport 80 -j SNAT --to-source 192.168.6.1
```

**MALANG**
```
iptables -N LOGGING
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j LOGGING
iptables -A LOGGING -m limit --limit 5/min -j LOG --log-prefix "iptables_FORWARD_denied: " --log-level 7
iptables -A LOGGING -j DROP
iptables -A INPUT -s 192.168.4.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 17:00 --timestop 23:59 -j ACCEPT
iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 00:00 --timestop 07:00 -j ACCEPT
iptables -A INPUT -s 192.168.0.0/24 -j REJECT
iptables -A INPUT -s 192.168.4.0/24 -j REJECT
```

**MOJOKERTO**
```
iptables -N LOGGING
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j LOGGING
iptables -A LOGGING -m limit --limit 5/min -j LOG --log-prefix "iptables_FORWARD_denied: " --log-level 7
iptables -A LOGGING -j DROP
```
