# Jarkom_Modul5_Lapres_D09

### 05111840000058 Rafif Ridho
### 05111840000128 Muhammad Rivadhli Purnomo

#### Kelompok kami pembagian subnet menggunakan metode vlsm

Pembagian subnet
![img](/img/1.jpg)

Jumlah IP pada tiap-tiap subnet dan range-nya yang ada pada topologi
![img](/img/2.jpg)

Kemudian untuk pembagian pohonnya adalah(puncaknya menggunakan prefix /22 karena menggunakan prefix /23 tidak dapat digunakan untuk membagi 2 subnet /24
![img](/img/3.jpg)

Membuat topologi jaringan sesuai dengan rancangan yang diberikan pada gambar di atas. Sekalian membuat bye.shnya.

### Topologi.sh

```
xterm -T KEDIRI -e linux ubd0=KEDIRI,jarkom umid=KEDIRI eth0=daemon,,,switch3 e$
xterm -T BATU -e linux ubd0=BATU,jarkom umid=BATU eth0=daemon,,,switch4 eth1=da$

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 m$
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,$
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch1 m$
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=dae$

#Klien
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,swi$
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch5 m$
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

Kemudian untuk kofigurasi DHCP Server pada MOJOKERTO dan DHCP RELAY pada KEDIRI, SURABAYA, dan BATU adalah
