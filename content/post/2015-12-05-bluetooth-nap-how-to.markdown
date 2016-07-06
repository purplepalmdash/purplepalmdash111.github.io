---
categories: ["Embedded"]
comments: true
date: 2015-12-05T20:03:41Z
title: Bluetooth NAP How-To
url: /2015/12/05/bluetooth-nap-how-to/
---

### Notice
Notice, this way is only for Bluez-5!!!!    

Refers to:    
[https://wiki.gentoo.org/wiki/Bluetooth_Network_Aggregation_Point](https://wiki.gentoo.org/wiki/Bluetooth_Network_Aggregation_Point)   
[https://wiki.gentoo.org/wiki/Bluetooth](https://wiki.gentoo.org/wiki/Bluetooth)     
[http://blog.fraggod.net/2015/03/28/bluetooth-pan-network-setup-with-bluez-5x.html](http://blog.fraggod.net/2015/03/28/bluetooth-pan-network-setup-with-bluez-5x.html)    

Bluez-4 could be much more easier via `pand`. For example, in ubuntu you could
setup the bluetooth PAN via:   

[http://blog.sumostyle.net/2009/11/ubuntu-tethering-via-bluetooth-pan/](http://blog.sumostyle.net/2009/11/ubuntu-tethering-via-bluetooth-pan/)   

### RF-KILL
Use rfkill for unblock the soft-blocked bluetooth adapter:     

```
[root@xxxx dash]# rfkill list bluetooth
2: hp-bluetooth: Bluetooth
        Soft blocked: yes
        Hard blocked: no
5: hci1: Bluetooth
        Soft blocked: yes
        Hard blocked: no
[root@xxxx dash]# rfkill unblock bluetooth

# rfkill list bluetooth
2: hp-bluetooth: Bluetooth
        Soft blocked: no
        Hard blocked: no
5: hci1: Bluetooth
        Soft blocked: no
        Hard blocked: no
```

### bluetoothctl
Use bluetoothctl for configurating the bluetooth adapter:    

```
[root@xxxx ]# bluetoothctl 
[NEW] Controller 40:2C:xx:xx:xx:xx xxxx #2 [default]
[bluetooth]# power on
Changing power on succeeded
[bluetooth]# discoverable on
Changing discoverable on succeeded
[bluetooth]# agent on
Agent registered
[bluetooth]# scan on
```

### PAN
Refers to:    
[http://blog.fraggod.net/2015/03/28/bluetooth-pan-network-setup-with-bluez-5x.html](http://blog.fraggod.net/2015/03/28/bluetooth-pan-network-setup-with-bluez-5x.html)    

```
--- machine-1
% bluetoothctl
[NEW] Controller 00:02:72:XX:XX:XX malediction [default]
[bluetooth]# power on
Changing power on succeeded
[CHG] Controller 00:02:72:XX:XX:XX Powered: yes
[bluetooth]# discoverable on
Changing discoverable on succeeded
[CHG] Controller 00:02:72:XX:XX:XX Discoverable: yes
[bluetooth]# agent on
...

--- machine-2 (snipped)
% bluetoothctl
[NEW] Controller 00:02:72:YY:YY:YY rpbox [default]
[bluetooth]# power on
[bluetooth]# scan on
[bluetooth]# agent on
[bluetooth]# pair 00:02:72:XX:XX:XX
[bluetooth]# trust 00:02:72:XX:XX:XX
```

Then Download the script from:    

[https://github.com/mk-fg/fgtk/blob/master/bt-pan](https://github.com/mk-fg/fgtk/blob/master/bt-pan)    

This `bt-pan` will setup both the server and the client.    

In Server:    

```
$ bt-pan --debug server $br
```

While the `$br` could be setup via following command(Take Gentoo for example):    

```
# vim /etc/conf.d/net

...
# Comment out this line, and add the following lines:
#config_eth0="dhcp"
config_eth0="null"
bridge_br1="eth0"
config_br1="dhcp"
# Next two lines, to make two values work (keep setfd before stp):
brctl_br1="setfd 1
stp on"
...
# ln -s net.lo /etc/init.d/net.br1
# rc-service net.eth0 stop && rc-service net.br1 start
# rc-update add net.br1 default 
```

In the Client side, do following:    

```
$ bt-pan client 00:02:72:XX:XX:XX
```

Now check the both client and server, you will find `bnep0` interface has been
created, you could see it via `ifconfig bnep0`.    

Assign the same IP network range address to the client side, as in the br1
side in server. After that you could ping each other via bluetooth!     
