# Membuat AP di Linux Menggunakan hostapd

## Persyaratan
- Virtual Machine ([VirtualBox](https://www.virtualbox.org/))
- OS Linux ([Ubuntu 24.04](https://releases.ubuntu.com/noble/))
- Akses root (`sudo`)
- Adapter Wi-Fi yang mendukung mode AP
- Network Bridged
- Koneksi internet

## Instalasi

```bash
sudo apt-get update
sudo apt-get install hostapd isc-dhcp-server iw iptables wget
```

## Langkah-Langkah

#### 1. Cek interface:

```bash
ip l show
```

![](https://github.com/fixploit03/catatan-linux/blob/main/Membuat%20AP%20di%20Linux%20Menggunakan%20hostapd/img/1.png)

Pastikan ada output interface wireless dan interface internet:
- Interface wireless: `wlan*, wlp*, wlx*`
- Interface internet: `eth*`, `enp*`

#### 2. Cek koneksi internet di interface internet:

```bash
ip a show [interface_internet]
ip route | grep default
ping -I [interface_internet] -c 4 google.com
```

![](https://github.com/fixploit03/catatan-linux/blob/main/Membuat%20AP%20di%20Linux%20Menggunakan%20hostapd/img/2.png)

#### 3. Cek apakah interface wireless mendukung mode AP:

```bash
iw list | grep -A 10 "Supported interface modes"
```

![](https://github.com/fixploit03/catatan-linux/blob/main/Membuat%20AP%20di%20Linux%20Menggunakan%20hostapd/img/3.png)

#### 4. Cek jenis cipher yang didukung interface wireless:

```bash
iw list | grep -A 12 "Supported Ciphers"
```

![](https://github.com/fixploit03/catatan-linux/blob/main/Membuat%20AP%20di%20Linux%20Menggunakan%20hostapd/img/4.png)

#### 5. Stop service yang dapat mengganggu:

```bash
sudo systemctl stop wpa_supplicant
sudo systemctl stop NetworkManager
```

#### 6. Set regulasi domain ke negara Indonesia:

```bash
sudo iw reg set ID
```

#### 7. Set IP address:

```bash
sudo ip a f dev [interface_wireless]
sudo ip a a 10.10.10.1/24 dev [interface_wireless]
sudo ip l s [interface_wireless] up
```

#### 8. Aktifkan internet sharing (NAT):

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

#### 9. Backup file `dhcpd.conf`:

```bash
sudo touch /var/lib/dhcp/dhcpd.leases
sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.orig
```

#### 10. Buat konfigurasi DHCP Server:

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

#### 11. Ubah permission file `dhcpd.leases`:

```bash
sudo chmod 666 /var/lib/dhcp/dhcpd.leases
```

#### 12. Jalankan DHCP Server:

```bash
sudo dhcpd -4 -cf /etc/dhcp/dhcpd.conf [interface_wireless]
```

![](https://github.com/fixploit03/catatan-linux/blob/main/Membuat%20AP%20di%20Linux%20Menggunakan%20hostapd/img/5.png)

#### 13. Download file konfigurasi `hostapd.conf`:

```
wget https://raw.githubusercontent.com/fixploit03/catatan-linux/refs/heads/main/Membuat%20AP%20di%20Linux%20Menggunakan%20hostapd/hostapd.conf
```

#### 14. Jalankan AP:

```bash
sudo hostapd -i [interface_wireless] -B hostapd.conf
```

![](https://github.com/fixploit03/catatan-linux/blob/main/Membuat%20AP%20di%20Linux%20Menggunakan%20hostapd/img/6.png)
