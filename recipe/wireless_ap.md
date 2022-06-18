# Raspberry pi 3B 設定 wifi ap mode 步驟記錄

- $ sudo apt update
- $ sudo apt install dnsmasq hostapd
- $ sudo systemctl stop dnsmasq
- $ sudo systemctl stop hostapd

### 設定固定 ip

- $ sudo vim /etc/dhcpcd.conf

```
interface wlan0
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
```

- $ sudo service dhcpcd restart

### 設定 DHCP server

- $ sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.bak
- $ sudo vim /etc/dnsmasq.conf

```
interface=wlan0
  dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
```

- $ sudo systemctl reload dnsmasq

### hostapd 設定

- $ sudo vim /etc/hostapd/hostapd.conf

```
country_code=TW
interface=wlan0
driver=nl80211
ssid=myPI
hw_mode=g
channel=11
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=12345678
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

- $ sudo vim /etc/default/hostapd

```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

- $ sudo systemctl unmask hostapd
- $ sudo systemctl enable hostapd
- $ sudo systemctl start hostapd

### 設定 routing

- $ sudo vim /etc/sysctl.conf

```
net.ipv4.ip_forward=1
```

- $ sudo apt install iptables # 如果沒有安裝 iptables 才需要執行。
- $ sudo iptables -t nat -A POSTROUTING -o usb0 -j MASQUERADE
- $ sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
- $ sudo iptables-restore < /etc/iptables.ipv4.nat

### Other

#### 分享乙太網路訊號

- $ sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

#### 分享手機 usb 共享網路

- $ sudo iptables -t nat -A POSTROUTING -o usb0 -j MASQUERADE
