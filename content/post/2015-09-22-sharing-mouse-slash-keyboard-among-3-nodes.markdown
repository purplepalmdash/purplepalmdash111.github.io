---
categories: ["Linux"]
comments: true
date: 2015-09-22T20:42:04Z
title: Sharing Mouse/Keyboard among 3 Nodes
url: /2015/09/22/sharing-mouse-slash-keyboard-among-3-nodes/
---

I have 3 computers which runs ArchLinux/Ubuntu15.04 Mate, they both connected
to the same ethernet, I'd like to use only one mouse/keyboard for controlling
them, following are the steps for how-to.      

### Network Configuration
The 3 computer are listed as following:    
* 192.168.1.11/dashArch/ArchLinux
* 192.168.1.13/dash32G/Ubuntu15.04
* 192.168.1.10/dashMate/Ubuntu15.04

Edit the `/etc/hosts` in 192.168.1.11:    

```
$ sudo vim /etc/hosts
192.168.1.13    dash32G
192.168.1.14    dashMate
```

In 192.168.1.13, Edit `/etc/hosts`:    

```
$ sudo vim /etc/hosts
192.168.1.11    dashArch
```

### Synergy Server
I use ArchLinux for Synergy Server, install it simply via:    

```
$ sudo pacman -S synergy
```

Run synergy via `synergy`, now configure it:    

![/images/2015_09_22_20_47_11_692x530.jpg](/images/2015_09_22_20_47_11_692x530.jpg)   

By drag a new machine in the configuration window, named dash32G:    

![/images/2015_09_22_21_00_13_732x508.jpg](/images/2015_09_22_21_00_13_732x508.jpg)    

Also drag a new machine in the configuraitn window, named dashMate:    

![/images/2015_09_22_22_35_02_607x363.jpg](/images/2015_09_22_22_35_02_607x363.jpg)   

Save the conf file as `~/SynergyArch.conf`, then configure the systemctl.    

Start the Synergy Server at the startup:    

```
$ cat /usr/lib/systemd/synergys@.service 
[Unit]
Description=Synergy Server Daemon
After=network.target

[Service]
User=%i
ExecStart=/usr/bin/synergys --no-daemon --config /home/dash/SynergyArch.conf --enable-crypto
Restart=on-failure

[Install]
WantedBy=multi-user.target
$ sudo systemctl start synergys@dash
$ sudo systemctl enable synergys@dash
Created symlink from
/etc/systemd/system/multi-user.target.wants/synergys@dash.service to
/usr/lib/systemd/system/synergys@.service.
```


### Synergy Client
Install synergy client on Ubuntu via:    

```
$ sudo apt-get install -y synergy
```

Configure the synergy as the client mode, like following:    

![/images/2015_09_22_21_18_42_697x546.jpg](/images/2015_09_22_21_18_42_697x546.jpg)    

Save the configuration file under your home directory, named it as
`synergyconfig.conf`.   

Since the Ubuntu15.04 Mate use lightdm for login, we add following line into
lightdm's configuration file:    

```
$ sudo vim /usr/share/lightdm/lightdm.conf.d/50-ubuntu-mate.conf
+ greeter-setup-script=/usr/bin/synergyc 192.168.1.11
```

The same configuration should be applied to dashMate.    

### Auto-login Synergyc Configuration
Start synergy after login:    

To start Synergy once you have logged into your X-session.   

```
Main Menu - System - Preferences - Personnel - Startup Applications   
[ Add ]  
  Name:     synergys
  Command:  synergys --config ~/.synergy.conf
  Comment:  synergys  
```
Also you should configure the mate configuration file:     

```
$ cat /usr/share/lightdm/lightdm.conf.d/50-ubuntu-mate.conf
[SeatDefaults]
user-session=mate
greeter-setup-script=/usr/bin/synergyc 192.168.1.11
autologin-user=xxxxxx
```
Now you will have the synergy runs after you login to the mate desktop.    
### Conclusion
Now restart the computer, and everytime you could switch from different
machines.   
