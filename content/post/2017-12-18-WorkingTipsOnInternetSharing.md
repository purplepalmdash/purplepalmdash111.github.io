+++
title = "WorkingTipsOnInternetSharing"
date = "2017-12-18T15:08:12+08:00"
description = "WorkingTipsOnInternetSharing"
keywords = ["Linux"]
categories = ["Linux"]
+++
### AIM
To set up an free wireless solution working for WebDuino.    

Because I located in china mainland, while our internet were forbidden by
government(Fuck GFW!), so I have to find another way for accross the firewall
and let WebDuino fetch back its updates.    

### Hardware Environment
Laptop.    
Wireless dongle(Fast)    

```
Bus 001 Device 002: ID 0bda:8179 Realtek Semiconductor Corp. RTL8188EUS 802.11n Wireless Network Adapter
```

### VirtualMachine
I set a virtual machine which runs in virtualbox. First I thought
configuration via gui would be easier, but later I found it's even inpossible
to use the gui configuration tools to let my usb wireless card acts as an ap.

Later I will use an ubuntu 32-bit server(i386) for setting up this wireless
ap.    

To specify usb wireless dongle to virtualbox's vm(Ubuntu16.04), do following:    

![/images/2017_12_18_15_15_58_749x283.jpg](/images/2017_12_18_15_15_58_749x283.jpg)

Then in your vm you will see the attached usb dongle.    

### System Configuration
#### Enable naming
If you didn't specify the naming, Ubuntu16.04(or later than Ubuntu12.04) won't
recognize your ethernet card or wireless card as "eth0" or "wlan0", so you
have to changint then in grub parameters:    

If you don't do this step, your wireless card will be recognized as something
like "wlx000e8e22xxxxxx", this will makes your configuration a little bit
confusing.    

```
# vim /etc/default/grub
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
# sudo update-grub
# sudo reboot
```
After rebooting, your wireless card will be recoginzed as "wlan0".    

Don't forget to update your `/etc/network/interface`, change from "enp0s" to
"eth".    

#### 8188eu hostapd
The system's default hostapd won't be used, we have to use 8188eu's own
hostapd for working, the steps are listed as following:    

```
$ sudo apt-get autoremove hostapd
$ wget https://github.com/jenssegers/RTL8188-hostapd/archive/v2.0.tar.gz
$ tar -zxvf v2.0.tar.gz
$ cd RTL8188-hostapd-2.0/hostapd
$ sudo make
$ sudo make install
$ sudo systemctl enable hostapd
$ sudo service hostapd restart
```
If you didn't see your hostapd working, simply reboot your machine, and check
again.    

#### hostapd configuration
Your hostapd configuration file is located in `/etc/hostapd/hostapd.conf`,
following are the configuration example:    

```
interface=wlan0
ssid=sosowifi
channel=1
#bridge=br0

# WPA and WPA2 configuration

macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=3
wpa_passphrase=XXXXXXXXXXXXXXXXX
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP

# Hardware configuration

driver=rtl871xdrv
ieee80211n=1
hw_mode=g
device_name=RTL8192CU
manufacturer=Realtek
```

Check the status via:    

```
$ sudo systemctl status hostapd
● hostapd.service - LSB: Advanced IEEE 802.11 management daemon
   Loaded: loaded (/etc/init.d/hostapd; bad; vendor preset: enabled)
   Active: active (running) since 一 2017-12-18 15:00:02 CST; 27min ago
     Docs: man:systemd-sysv-generator(8)
  Process: 1502 ExecStart=/etc/init.d/hostapd start (code=exited, status=0/SUCCESS)
    Tasks: 1 (limit: 512)
   CGroup: /system.slice/hostapd.service
           └─1525 /usr/local/bin/hostapd -B -P /var/run/hostapd.pid /etc/hostapd/hostapd.conf
```
#### wlan0 configuration
Setup the wlan0 ip address via following:    


```
# vim /etc/network/interface
	# wireless wlan0
	allow-hotplug wlan0
	iface wlan0 inet static
	address 10.0.70.1
	netmask 255.255.255.0
```
We set wlan0's ip address as the gateway for our wifi-network.   

#### dhcpcd
We have to use dhcpd for assiging the ip address to connected.    

```
$ sudo  apt-get install  isc-dhcp-server
$ sudo vim /etc/dhcp/dhcpd.conf
ddns-update-style none;
ignore client-updates;
authoritative;
option local-wpad code 252 = text;
 
subnet 10.0.70.0 netmask 255.255.255.0 {
# --- default gateway
option routers 10.0.70.1;
# --- Netmask
option subnet-mask 255.255.255.0;
# --- Broadcast Address
option broadcast-address 10.0.70.255;
# --- Domain name servers, tells the clients which DNS servers to use.
option domain-name-servers
10.0.70.1, 8.8.8.8, 8.8.4.4;
option time-offset 0;
range 10.0.70.3 10.0.70.13;
default-lease-time 1209600;
max-lease-time 1814400;
}
```
We won't add dhcpd for auto-startup, because we don't want person to connect
to our ap at the very beginning time.    

#### IP Forwarding
We enable the ip forwarding of the kernel and setup the postrouting for
iptables:    

```
$ sudo iptables -t nat -A POSTROUTING -s 10.0.70.0/24 ! -d 10.0.70.0/24  -j MASQUERADE
$ sudo dhcpd wlan0
$ sudo bash
# echo 1 > /proc/sys/net/ipv4/ip_forward
```
Now you should be able to access the network.    

### Trouble-Shooting
If you use virtualbox's redirect usb. you will encounter several problems. Try
to change from virtualbox to virt-manager:    

```
# VBoxManage clonehd --format RAW UbuntuServer.vdi UbuntuServer.img
# qemu-img convert -f raw UbuntuServer.img -O qcow2 UbuntuServer.qcow2
```
Now create a new virtual machine in virt-manager, you will find your wifi
sharing become stable.    

Redirect the usb device via spice driver.    
