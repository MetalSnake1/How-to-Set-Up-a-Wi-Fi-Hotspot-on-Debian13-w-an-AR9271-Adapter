Important Note: You must have an ethernet connection running to the Debian13 computer. 

The AR9271 uses a driver called ath9k_htc, so you'll have to install the firmware package with: 
```bash
sudo apt install firmware-ath9k-htc
```

## Software Setup:
*hostapd - creates the wireless access point
*dnsmasq - provides DHCP and DNS services
*iptables - handles NAT for internet sharing.

### Install: 
```bash
sudo apt install firmware-ath9k-htc
```

Okay, now when I used the command
```bash
sudo iwconfig
```

My AR9271 adapters interface name was: wlx24ec99a67a4e

## Hostapd Setup

So, you'll want to nano into /etc/hostapd/hostapd.conf and insert this: (you can change the password)
```insertion
interface=wlx24ec99a67a4e driver=nl80211 ssid=DebianHotspot hw_mode=g channel=7 wmm_enabled=1 auth_algs=1 wpa=2 wpa_passphrase=YOUR_PASSWORD wpa_key_mgmt=WPA-PSK rsn_pairwise=CCMP
```
That configures hostapd, which is the program that turns your standard wifi adapter into an AP. The configuration that I gave you tells hostapd which wifi adapter to use, which network to create, and how clients should authenticate. 

## Dnsmasq Configuration 
It's important to make sure the AR9271 hotspot uses a separate network from the ethernet connection, in order to prevent conflicts between the two interfaces. 
```bash
sudo ip addr flush dev wlx24ec99a67a4e
sudo ip addr add 192.168.1.1/24 dev wlx24ec99a67a4e
sudo ip link set wlx24ec99a67a4e up
```
This will ensure that there are no conflicts. 

Next edit /etc/dnsmasq.conf with: 
```bash
interface=wlx24ec99a67a4e
bind-interfaces

dhcp-range=192.168.1.10,192.168.1.100,12h

dhcp-option=3,192.168.1.1
dhcp-option=6,192.168.1.1
```

After saving the file, run this command:
```bash
sudo systemctl restart dnsmasq
```
## Now Enable Ip Forwarding:
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```
To make it permanent after reboot: 
```bash
sudo nano /etc/sysctl.conf
```
Add this: net.ipv4.ip_forward=1

## Add NAT
Find out what your ethernet interface is. Now run: 
```bash
sudo iptables -t nat -A POSTROUTING -o <the actual name of your ethernet interface> -j MASQUERADE
```
(don't include < > in the interface name)
This makes it so the device that connects to the hotspot will actually have access to the internet, because the device can actually route back to the 192.168.1.x subnet 

## Allow traffic forwarding
This command allows traffic from the Wi-Fi adapter to reach the ethernet
```bash
sudo iptables -A FORWARD -i wlx24ec99a67a4e -o <the actual name of your ethernet interface> -j ACCEPT
```

Now allow return traffic back to the Wi-Fi clients
```bash
sudo iptables -A FORWARD -i <the actual name of your ethernet interface> -o wlx24ec99a67a4e -m state --state RELATED,ESTABLISHED -j ACCEPT
```

## (Optional) Make iptables rules survive reboot: 
Install persistence: 
```bash
sudo apt install iptables-persistent
```
Save your rules:
```bash
sudo netfilter-persistent save
```
