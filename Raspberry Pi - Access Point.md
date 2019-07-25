# Raspberry Pi - Access Point
---

## Description
This document outline a base instruction set for setting up a Raspberry Pi 2 with the following features

* Raspbian Jessie *(OS based on Debian Jessie)*
* Custom WIFI SSID

---

## Requirements

1. 1x [Raspberry Pi 2](https://www.adafruit.com/products/2358)
2. 1x [Micro SD Card](https://www.adafruit.com/products/2693) *(8GB minumum, 16GB preferred)*
3. 2x [USB Wifi module](https://www.adafruit.com/products/814) *(with RTl8192 drive)*

#### Setup Requirements

1. A computer (we used a Mac for this example)
2. Network connection (hardwire for Raspberry Pi)
3. Software
   * [ApplePi Baker](http://www.tweaking4all.com/?wpfb_dl=94)
   * [Raspbian Jesse](https://downloads.raspberrypi.org/raspbian_latest) *(latest)*
   * [iTerm 2](https://www.iterm2.com/) (or other terminal)

---
## Setup

### Initial
Once you have downloaded both the Raspbian image and ApplePi Baker

##### OSX
1. Run ApplePi Baker
2. Enter your administrative password
3. Select your SD Card
4. Select your IMG file in the *Pi-Ingredients: IMG Recipe seciton*
5. Click **'Restore Backup'**
6. Have a coffee, sit back, relax (it takes about 5 minutes)
7. Eject your SD Card

##### First Boot
1. Connect your Ethernet cable
2. Connect your Wifi Modules
3. Connect your mouse
4. Connect your keyboard
5. Connect your HDMI
6. Insert your newly flash SD Cards
7. Plug in power
8. First time boot take over a minute or two

#####Initial Run
1. Open terminal
2. We will be working as super user, so in terminal ```sudo su```
3. Update the Pi ```apt-get update```
4. Have another coffee
5. Upgrade the Pi ```apt-get upgrade``` (optional)
6. Configure your Pi ```raspi-config```
   * expand root partition
   * enable ssh 
   * save / exit
7. Once it's all done ```reboot now```

### Access Point Setup
We will be working as super user, so in terminal 
```
sudo su
apt-get install hostapd isc-dhcp-server
```

```
nano /etc/dhcp/dhcpd.conf
```

* comment out ```option domain-name "example.org";``` 
* comment out ```option domain-name-servers ns1.example.org, ns2.example.org;```
* uncomment ```#authoritative;```
* add the following to the **bottom** 

```
subnet 192.168.42.0 netmask 255.255.255.0 {
	range 192.168.42.10 192.168.42.50;
	option broadcast-address 192.168.42.255;
	option routers 192.168.42.1;
	default-lease-time 600;
	max-lease-time 7200;
	option domain-name "local";
	option domain-name-servers 8.8.8.8, 8.8.4.4;
}	
```

* save the file (ctrl+x, type Y)
* ```nano /etc/default/isc-dhcp-server``` 
  * replace ```INTERFACES=""``` with ```INTERFACES="wlan1"```	* save the file (ctrl+x, type Y)

##### wlan1 as static IP
1. ```ifdown wlan1```
2. ```nano /etc/network/interfaces```
3. replace the whole file with

```
# interfaces(5) file used by ifup(8) and ifdown(8)

# Please note that this file is written to be used with dhcpcd
# For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'

# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d

auto lo
iface lo inet loopback

iface eth0 inet manual

allow-hotplug wlan0
iface wlan0 inet manual
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

# ---
# ORIGINAL
# ---	
# allow-hotplug wlan1
# iface wlan1 inet manual
#    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

# ---
# EDITED
# ---
allow-hotplug wlan1
iface wlan1 inet static
  address 192.168.42.1
  netmask 255.255.255.0

# ---
# BRIDGE
# ---

#iface br0 inet static
#  bridge_ports wlan0 wlan1
#  address 192.168.1.11
#  netmask 255.255.255.0
#  network 192.168.1.0
#  gateway 192.168.1.2

up iptables-restore < /etc/iptables.ipv4.nat  
```
1. ctrl+x y
2. ```sudo ifconfig wlan1 192.168.42.1```

### Configure Point Setup
1. ```nano /etc/hostapd/hostapd.conf```
2. we named our SSID **Starbucks**

```
interface=wlan0
driver=rtl871xdrv
hw_mode=g
interface=wlan1
driver=rtl871xdrv
ssid=Starbucks
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
bridge=br0
```
1. ctrl+x y
2. ```nano /etc/default/hostapd```
3. uncomment and edit ```#DAEMON_CONF=""``` to ```DAEMON_CONF="/etc/hostapd/hostapd.conf"```
4. ctrl+x y

### Configure NAT (Network Address Translation)
1. ```nano /etc/sysctl.conf```
2. add ```net.ipv4.ip_forward=1``` to the bottom
3. ctrl+x y
4. ```sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"```

### iptables rules

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
iptables -A FORWARD -i wlan0 -o wlan1 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i wlan1 -o wlan0 -j ACCEPT
```

automate on reboot ```sh -c "iptables-save > /etc/iptables.ipv4.nat"```

### Updatehostapd

```
wget http://adafruit-download.s3.amazonaws.com/adafruit_hostapd_14128.zip
unzip adafruit_hostapd_14128.zip
mv /usr/sbin/hostapd /usr/sbin/hostapd.ORIG
mv hostapd /usr/sbin
chmod 755 /usr/sbin/hostapd
/usr/sbin/hostapd /etc/hostapd/hostapd.conf
```

ctrl+c

### daemon
```
service hostapd start 
service isc-dhcp-server start
update-rc.d hostapd enable 
update-rc.d isc-dhcp-server enable
```
