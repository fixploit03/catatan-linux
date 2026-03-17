# Membuat AP di Linux Menggunakan hostapd

## Persyaratan
- OS Linux ([Ubuntu 24.04](https://releases.ubuntu.com/noble/))
- Akses root (`sudo`)
- Adapter Wi-Fi yang mendukung mode AP
- Koneksi internet

## Instalasi

```bash
sudo apt-get update
sudo apt-get install hostapd isc-dhcp-server iw
```

## Langkah-Langkah

#### Cek interface:

```bash
ip l show
```

Pastikan ada output interface wireless dan interface internet:
- Interface wireless: `wlan*, wlp*, wlx*`
- Interface internet: `eth*`, `enp*`

#### Cek koneksi internet di interface internet:

```bash
ip a show [interface_internet]
ip route | grep default
ping -I [interface_internet] -c 4 google.com
```

#### Cek apakah interface wireless mendukung mode AP:

```bash
iw list | grep -A 10 "Supported interface modes"
```

#### Cek jenis cipher yang didukung interface wireless:

```bash
iw list | grep -A 12 "Supported Ciphers"
```

#### Stop service yang dapat mengganggu:

```bash
sudo systemctl stop wpa_supplicant
sudo systemctl stop NetworkManager
```

#### Set regulasi domain ke negara Indonesia:

```bash
sudo iw reg set ID
```

#### Set IP address:

```bash
sudo ip a f dev [interface_wireless]
sudo ip a a 10.10.10.1/24 dev [interface_wireless]
sudo ip l s [interface_wireless] up
```

#### Aktifkan internet sharing (NAT):

```bash
# Bersihkan semua rules iptables yang ada
sudo iptables -F
sudo iptables -t nat -F

# Aktifkan IP Forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Aktifkan NAT (Masquerade) pada interface internet
sudo iptables -t nat -A POSTROUTING -o [interface_internet] -j MASQUERADE
# Izinkan paket dari interface wireless diteruskan ke internet
sudo iptables -A FORWARD -i [interface_wireless] -o [interface_internet] -j ACCEPT
# Izinkan paket balik dari internet masuk kembali ke interface wireless
sudo iptables -A FORWARD -i [interface_internet] -o [interface_wireless] -m state --state RELATED,ESTABLISHED -j ACCEPT
```

#### Backup file `dhcpd.conf`:

```bash
sudo touch /var/lib/dhcp/dhcpd.leases
sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.orig
```

#### Buat konfigurasi DHCP Server:

```bash
sudo tee /etc/dhcp/dhcpd.conf <<EOF
# DNS Server
option domain-name-servers 8.8.8.8, 8.8.4.4;

# Waktu sewa IP address
default-lease-time 600;
max-lease-time 7200;

# Konfigurasi jaringan
subnet 10.10.10.0 netmask 255.255.255.0 {
	# Range IP address
	range 10.10.10.2 10.10.10.254;
	# IP gateway
	option routers 10.10.10.1;
	# IP Broadcast
	option broadcast-address 10.10.10.255;
	# Netmask
	option subnet-mask 255.255.255.0;
}
EOF
```

#### Ubah permission file `dhcpd.leases`:

```bash
sudo chmod 666 /var/lib/dhcp/dhcpd.leases
```

#### Jalankan DHCP Server:

```bash
sudo dhcpd -4 -cf /etc/dhcp/dhcpd.conf [interface_wireless]
```

#### Jalankan AP:

```bash
sudo hostapd -i [interface_wireless] -B hostapd.conf
```
