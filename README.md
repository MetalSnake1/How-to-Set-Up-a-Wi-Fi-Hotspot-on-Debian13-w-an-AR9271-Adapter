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

So, you'll want to nano into /etc/hostapd/hostapd.conf and insert this:
```insertion
interface=wlx24ec99a67a4e driver=nl80211 ssid=DebianHotspot hw_mode=g channel=7 wmm_enabled=1 auth_algs=1 wpa=2 wpa_passphrase=YOUR_PASSWORD wpa_key_mgmt=WPA-PSK rsn_pairwise=CCMP
```
That configures hostapd, which is the program that turns your standard wifi adapter into an AP. The configuration that I gave you tells hostapd which wifi adapter to use, which network to create, and how clients should authenticate. 
