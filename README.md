# Jarkom_Modul5_Lapres_D09

### 05111840000058 Rafif Ridho
### 05111840000128 Muhammad Rivadhli Purnomo

#### Kelompok kami pembagian subnet menggunakan metode vlsm

Pembagian subnet

![img](/img/1.png)

Jumlah IP pada tiap-tiap subnet dan range-nya yang ada pada topologi

![img](/img/2.jpg)

Kemudian untuk pembagian pohonnya adalah(puncaknya menggunakan prefix /22 karena menggunakan prefix /23 tidak dapat digunakan untuk membagi 2 subnet /24

![img](/img/3.jpg)

Pembagian NID tiap subnet 

![img](/img/3-1.jpg)

Membuat topologi jaringan sesuai dengan rancangan yang diberikan pada gambar di atas. Sekalian membuat bye.shnya.

### Topologi.sh

```
# switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &
uml_switch -unix switch4 > /dev/null < /dev/null &
uml_switch -unix switch5 > /dev/null < /dev/null &
uml_switch -unix switch6 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.78.41 eth1=daemon,,,switch3 eth2=daemon,,,switch4 mem=96M &
xterm -T KEDIRI -e linux ubd0=KEDIRI,jarkom umid=KEDIRI eth0=daemon,,,switch3 eth1=daemon,,,switch1 eth2=daemon,,,switch5 mem=96M &
xterm -T BATU -e linux ubd0=BATU,jarkom umid=BATU eth0=daemon,,,switch4 eth1=daemon,,,switch6 eth2=daemon,,,switch2 mem=96M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=128M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch1 mem=128M &
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch1 mem=128M &

#Klien
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch6 mem=96M &
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch5 mem=96M &
```

Pada semua UML router, ketikkan nano /etc/sysctl.conf serta uncomment net.ipv4.ip_forward=1 . Untuk mengaktifkan perubahan baru ketikkan sysctl -p.

Lalu setting pada nano /etc/network/interfaces pada masing-masing uml dengan konfigurasi

1. Surabaya (Sebagai Router)
```
auto lo
iface lo inet loopback

#cloud
auto eth0
iface eth0 inet static
address 10.151.78.42
netmask 255.255.255.252
gateway 10.151.78.41

#switch3
auto eth1
iface eth1 inet static
address 192.168.0.1
netmask 255.255.255.252

#switch4
auto eth2
iface eth2 inet static
address 192.168.0.5
netmask 255.255.255.252
```

2. Batu (Sebagai Router)
```
auto lo
iface lo inet loopback

#switch4
auto eth0
iface eth0 inet static
address 192.168.0.6
netmask 255.255.255.252
gateway 192.168.0.5

#switch6
auto eth1
iface eth1 inet static
address 192.168.2.1
netmask 255.255.255.0

#switch2
auto eth2
iface eth2 inet static
address 10.151.79.81
netmask 255.255.255.248
```

3. Kediri (Sebagai Router)
```
auto lo
iface lo inet loopback

#switch3
auto eth0
iface eth0 inet static
address 192.168.0.2
netmask 255.255.255.252
gateway 192.168.0.1

#switch1
auto eth1
iface eth1 inet static
address 192.168.0.9
netmask 255.255.255.248

#switch5
auto eth2
iface eth2 inet static
address 192.168.1.1
netmask 255.255.255.0
```

4. Madiun (Sebagai Web Server)
```
auto lo
iface lo inet loopback

#switch1
auto eth0
iface eth0 inet static
address 192.168.0.10
netmask 255.255.255.248
gateway 192.168.0.9
```

5. Probolinggo (Sebagai Web Server)
```
auto lo
iface lo inet loopback

#switch1
auto eth0
iface eth0 inet static
address 192.168.0.11
netmask 255.255.255.248
gateway 192.168.0.9
```

6. Malang (Sebagai DNS Server)
```
auto lo
iface lo inet loopback

#switch2
auto eth0
iface eth0 inet static
address 10.151.79.82
netmask 255.255.255.248
gateway 10.151.79.81
```

7. Mojokerto (Sebagai DNS Server)
```
auto lo
iface lo inet loopback

#switch2
auto eth0
iface eth0 inet static
address 10.151.79.83
netmask 255.255.255.248
gateway 10.151.79.81
```

Kemudian menambahkan file route.sh di Surabaya saja, karena pada uml untuk 0.0.0.0 sudah otomatis terbuat
```
#to Malang & Mojokerto via Batu
ip route add 10.151.79.80/29 via 192.168.0.6
#to Sidoarjo via Batu
ip route add 192.168.2.0/24 via 192.168.0.6
#to Gresik via Kediri
ip route add 192.168.1.0/24 via 192.168.0.2
#to Probolinggo & Madiun via Kediri
ip route add 192.168.0.8/29 via 192.168.0.2
```

Kemudian untuk kofigurasi DHCP Server pada MOJOKERTO dan DHCP RELAY pada KEDIRI, SURABAYA, dan BATU adalah

Pertama sebelum install harus melakukan apt-get update terlebih dahulu

### Pada DHCP Server Jalankan apt-get install isc-dhcp-server pada uml MOJOKERTO

Lalu setting pada nano /etc/dhcp/dhcpd.conf, dengan konfigurasi
```
subnet 10.151.79.80 netmask 255.255.255.248 {
     option routers 10.151.79.81;
     option routers 10.151.79.87;
}

#sidoarjo
subnet 192.168.2.0 netmask 255.255.255.0 {
    range 192.168.2.2 192.168.2.203;
    option routers 192.168.2.1;
    option domain-name-servers 10.151.79.82;
    option domain-name-servers 202.46.129.2;
    default-lease-time 600;
    max-lease-time 7200;
}

#gresik
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.2 192.168.1.213;
    option routers 192.168.1.1;
    option broadcast-address 192.168.1.1;
    option domain-name-servers 202.46.129.2;
    option domain-name-servers 10.151.79.82;
    default-lease-time 600;
    max-lease-time 7200;
}
```

Kemudian juga buka nano /etc/default/isc-dhcp-server, lalu konfigurasi "INTERFACESv4=eth0":

![img](/img/4.jpg)

#### Kendala yang dialami 
Untuk ip dinamisnya terjadi error pada client kami, maka dari itu untuk nomor selanjutnya kami menggunakan ip static seperti yang ada pada subnetting di atas 

## No1: Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi SURABAYA menggunakan iptables, namun Bibah tidak ingin kalian menggunakan MASQUERADE.

Pada UML SURABAYA Ketikkan iptables sebagai berikut :

```
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j SNAT --to-source 10.151.78.42
```

Mencoba ping google.com selain di SURABAYA

![img](/img/5.jpg)

## No2: Kalian diminta untuk mendrop semua akses SSH dari luar Topologi (UML) Kalian pada server yang memiliki ip DMZ (DHCP dan DNS SERVER) pada SURABAYA demi menjaga keamanan.

Pada UML SURABAYA Ketikkan iptables sebagai berikut :

```
iptables -A FORWARD -d 10.151.79.80/29 -i eth0 -p tcp --dport 22 -j DROP
```

![img](/img/6.jpg)

## No3: Karena tim kalian maksimal terdiri dari 3 orang, Bibah meminta kalian untuk membatasi DHCP dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari mana saja menggunakan iptables pada masing masing server, selebihnya akan di DROP.

Pada UML MALANG Ketikkan iptables sebagai berikut :

```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

Ping ke MALANG pada 4 UML berbeda, maka untuk UML ke-4 tidak akan bisa mengakses Malang karena sudah dibatasi menjadi 3 saja

![img](/img/7.jpg)

## No4: Membatasi akses ke MALANG, Akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin sampai Jumat.

Pada UML MALANG Ketikkan iptables sebagai berikut :

```
iptables -A INPUT -s 192.168.2.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -s 192.168.2.0/24 -j REJECT
```

Saat Selasa tanggal 29 jam 14:09:12
![img](/img/8.jpg)

Saat Selasa tanggal 29 jam 19:00:03
![img](/img/8-1.jpg)

## No5: Membatasi akses ke MALANG, Akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap harinya.

Pada UML MALANG Ketikkan iptables sebagai berikut :

```
iptables -A INPUT -s 192.168.1.0/24 -m time --timestart 07:00 --timestop 17:00 -j REJECT
```

Saat Selasa tanggal 29 jam 19:01:19
![img](/img/9.jpg)

Saat Selasa tanggal 29 jam 12:00:05
![img](/img/9-1.jpg)

## No6: SURABAYA disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada PROBOLINGGO port 80 dan MADIUN port 80.

Untuk Nomor 6 tidak kami kerjakan

## No7: Bibah ingin agar semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap UML yang memiliki aturan drop.

Pada UML SURABAYA, MALANG, dan MOJOKERTO

```
iptables -N LOGGING
iptables -A INPUT -j LOGGING
iptables -A OUTPUT -j LOGGING
iptables -A LOGGING -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -A LOGGING -j DROP
```

Apabila meminta request pada uml yang yang dituju, maka log requestnya akan ditampilkan di uml yang dituju
![img](/img/10.jpg)
