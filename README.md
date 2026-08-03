Important Note: You must have an ethernet connection running to the Debian13 computer. 
  


## Software Setup:
*hostapd - creates the wireless access point
*dnsmasq - provides DHCP and DNS services
*iptables - handles NAT for internet sharing.

### Install: 
```bash
sudo apt install firmware-ath9k-htc
```
and 

Also, that the AR9271 may require the non-free-firmware repository. If you get: Unable to locate package firmware-ath9k-htc, that's probably why.  

```bash
sudo apt install hostapd dnsmasq
```

Okay, now when I used the command
```bash
sudo iwconfig
```

I found that my AR9271 adapters interface name was: wlx24ec99a67a4e. Your interface name will most likely be different.

## Disable Network Manager:
Network manager will most likely try to reconnect the adapter to a network instead of allowing hostapd to control it. So, run:

```bash
sudo nmcli device set wlx24ec99a67a4e managed no
```

## Hostapd Setup

So, you'll want to nano into /etc/hostapd/hostapd.conf and insert this: (you can change the password)
```insertion
interface=wlx24ec99a67a4e driver=nl80211 ssid=DebianHotspot hw_mode=g channel=7 wmm_enabled=1 auth_algs=1 wpa=2 wpa_passphrase=YOUR_PASSWORD wpa_key_mgmt=WPA-PSK rsn_pairwise=CCMP
```
## Make sure to point hostapd at the config file.
Nano /etc/default/hostapd and add: 

```bash
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
Restart hostapd after editing it. Use 
```bash
sudo systemctl restart hostapd
```


Then start hostapd with
```bash
sudo systemctl enable hostapd
sudo systemctl start hostapd
```

That configures hostapd, which is the program that turns your standard wifi adapter into an AP. The configuration that I gave you tells hostapd which wifi adapter to use, which network to create, and how clients should authenticate. 

## Dnsmasq Configuration 
It's important to make sure the AR9271 hotspot uses a separate network from the ethernet connection, in order to prevent conflicts between the two interfaces. 
```bash
sudo ip addr flush dev wlx24ec99a67a4e
sudo ip addr add 192.168.1.1/24 dev wlx24ec99a67a4e
sudo ip link set wlx24ec99a67a4e up
```
*Quick note: "sudo ip addr add 192.168.1.1/24 dev wlx24ec99a67a4e" this must be run after every reboot, unless you make the ip persistent across every reboot.

These commands will remove any existing IP configuration from the Wi-Fi adapter and assign it the static address 192.168.1.1/24, which is the gateway for the hotspot clients.

Add or place the following lines in /etc/dnsmasq.conf with: 
```insertion
interface=wlx24ec99a67a4e
bind-interfaces

dhcp-range=192.168.1.10,192.168.1.100,12h

dhcp-option=3,192.168.1.1
dhcp-option=6,192.168.1.1
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
This makes it so outgoing packets appear to come from the Debian computers ethernet IP. Replies are translated back to the correct hotspot client. That allows devices on the 192.168.1.x subnet to access the internet. 

## Allow traffic forwarding
This command allows traffic from the Wi-Fi adapter to reach the ethernet
```bash
sudo iptables -A FORWARD -i wlx24ec99a67a4e -o <the actual name of your ethernet interface> -j ACCEPT
```

Now allow return traffic back to the Wi-Fi clients
```bash
sudo iptables -A FORWARD -i <the actual name of your ethernet interface> -o wlx24ec99a67a4e -m state --state RELATED,ESTABLISHED -j ACCEPT
```



## Start the service and use the hotspot

```bash
sudo systemctl enable hostapd dnsmasq
sudo systemctl restart hostapd
sudo systemctl restart dnsmasq
```

## Verify

```bash
sudo systemctl status hostapd
sudo systemctl status dnsmasq
```
Just check and see if both services are active.

## (Optional) Make iptables rules survive reboot: 
Install persistence: 
```bash
sudo apt install iptables-persistent
```
Save your rules:
```bash
sudo netfilter-persistent save
```
