---
categories: ["Linux"]
comments: true
date: 2015-08-31T12:07:37Z
title: Tips On Remove maas-dhcp
url: /2015/08/31/tips-on-remove-maas-dhcp/
---

### Solution
Previous I always encounter problems in removing maas-dhcp, so I checked some materials
show the result is because I disabled apparmor, the solution is:    

```
$ vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="security=selinux selinux=1  apparmor=1 security=apparmor"
```
Add apparmor related, and reboot the computer.    
Now you could remove the maas-dhcp and install new softwares.   

### TIPS
#### apparmor status
View apparmor status:    

```
sudo apparmor_status
```
#### Track Reason
Via restart you could know the exact reason:   

```
$ sudo /etc/init.d/apparmor restart
* Reloading AppArmor profiles * AppArmor not available as kernel LSM.
[fail]
```
The reason is because the LSM is not avaiable, by editing the grub configuration you
could enable apparmor again.   

### Disable checking in deb
Hold the package, then you could continue with installing other packages.   

```
# sudo apt-mark hold <package>
OR
# echo <package> hold | sudo dpkg --set-selections
```


