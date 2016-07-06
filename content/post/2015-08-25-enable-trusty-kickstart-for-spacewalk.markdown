---
categories: ["Linux"]
comments: true
date: 2015-08-25T16:06:17Z
title: Enable Trusty KickStart For SpaceWalk
url: /2015/08/25/enable-trusty-kickstart-for-spacewalk/
---

### Import distro-tree
Copy iso content into the distro-tree:    

```
# mount -t  iso9660 -o loop /mnt/iso/ubuntu-14.04.3-server-amd64.iso /mnt1/
# mkdir /var/distro-trees/ubuntu-14.04.3-amd64
# chmod 755 /var/distro-trees/ubuntu-14.04.3-amd64/
# cp -ar /mnt1/* /var/distro-trees/ubuntu-14.04.3-amd64/
```
We will copy the pxeboot startup file from CentOS7 into ubuntu14.04 distro tree for
cheating the spacewalk:    

```
# mount -t iso9660 -o loop /mnt/iso/CentOS-7-x86_64-Everything-1503-01.iso /mnt2/
# mkdir -p /var/distro-trees/ubuntu-14.04.3-amd64/images/pxeboot/
# cp /mnt2/images/pxeboot/{initrd.img,vmlinuz} /var/distro-trees/ubuntu-14.04.3-amd64/images/pxeboot/
# ls /var/distro-trees/ubuntu-14.04.3-amd64/images/pxeboot/ -l -h
....
```

### Kickstarting
Now in spacewalk go to Systems->Kickstart->Distributions, with the parameters like
following picture:    

![/images/2015_08_25_16_17_53_595x352.jpg](/images/2015_08_25_16_17_53_595x352.jpg)    
We copy the CentOS Kernel and initrd, so Spacewalk use redhat parameters for this
distribution, later we will fix this.     

Now create a profile via go to Systems->Kickstart->Profiles and "upload a new kickstart
file", just like following picture.    

![/images/2015_08_25_16_22_53_617x590.jpg](/images/2015_08_25_16_22_53_617x590.jpg)   

We fix the hardcoded breed from redhat to ubuntu via following steps:    

Get the list info via `cobbler list`:    

![/images/2015_08_25_16_27_12_515x205.jpg](/images/2015_08_25_16_27_12_515x205.jpg)    

Report the selected distro info:    

```
# cobbler distro report --name=trustyamd64:1:SpacewalkDefaultOrganization
```

![/images/2015_08_25_16_28_48_990x256.jpg](/images/2015_08_25_16_28_48_990x256.jpg)    

Edit it like following:    

```
# cobbler distro edit --name trustyamd64:1:SpacewalkDefaultOrganization --breed=ubuntu --os-version=jaunty --initrd=/var/distro-trees/ubuntu-14.04.3-amd64/install/netboot/ubuntu-installer/amd64/initrd.gz --kernel=/var/distro-trees/ubuntu-14.04.3-amd64/install/vmlinuz 
```

We should notice the initrd file and kernel has been modified!!!   

![/images/2015_08_25_16_32_24_933x261.jpg](/images/2015_08_25_16_32_24_933x261.jpg)   

Now edit the `/var/lib/tftpboot/pxelinux.cfg/default` file, find the item and modify it like following:    

```
LABEL trustykickstart:1:SpacewalkDefaultOrganization
        kernel /images/trustyamd64:1:SpacewalkDefaultOrganization/vmlinuz
        MENU LABEL trustykickstart:1:SpacewalkDefaultOrganization
        append initrd=/images/trustyamd64:1:SpacewalkDefaultOrganization/initrd.gz  ks=http://192.168.0.79/trustykickstart.cfg ksdevice=eth0 --
        ipappend 2
```
Now you could kickstart your ubuntu trusty now.     

### Register to SpaceWalk
First create a key under spacewalk for activating all of the trusty clients:    
![/images/2015_08_26_11_10_35_378x489.jpg](/images/2015_08_26_11_10_35_378x489.jpg)     

Register the client via:    

```
# apt-get install apt-transport-spacewalk rhnsd
# vim /usr/lib/python2.7/xmlrpclib.py
    def dump_nil (self, value, write):
    - if not self.allow_none:
    - raise TypeError, "cannot marshal None unless allow_none is enabled"
    +# if not self.allow_none:
    +# raise TypeError, "cannot marshal None unless allow_none is enabled"
# apt-get install python-libxml2
# mkdir /var/lock/subsys
# vim /etc/hosts
10.11.11.3      spacewalk
# rhnreg_ks --activationkey=1-trustyamd64 --serverUrl=http://spacewalk/XMLRPC
Warning: unable to enable rhnsd with chkconfig
```
Register to the main channel:    

```
# cat /etc/apt/sources.list.d/spacewalk.list
deb spacewalk://spacewalk channels: main
# apt-get update
```

Added more software channels in the spacewalk web.    

![/images/2015_08_26_11_22_13_607x454.jpg](/images/2015_08_26_11_22_13_607x454.jpg)   

Update it via:   

```
# apt-get update && apt-get update
Apt-Spacewalk: Updating sources.list
WARNING:root:could not open file '/etc/apt/sources.list'

Ign spacewalk://spacewalk channels: InRelease
Ign spacewalk://spacewalk channels: Release.gpg
Ign spacewalk://spacewalk channels: Release
Ign spacewalk://spacewalk channels:/main amd64 Packages/DiffIndex
Ign spacewalk://spacewalk channels:/main i386 Packages/DiffIndex
Get:1 spacewalk://spacewalk channels:/trusty-amd64-updates amd64 Packages [829 kB]
Get:2 spacewalk://spacewalk channels:/trusty-amd64-backports amd64 Packages [5,177 B]
Get:3 spacewalk://spacewalk channels:/trusty-amd64-security amd64 Packages [437 kB]
Get:4 spacewalk://spacewalk channels:/trusty-amd64-updates i386 Packages [829 kB]
Get:5 spacewalk://spacewalk channels:/trusty-amd64-backports i386 Packages [5,177 B]
Get:6 spacewalk://spacewalk channels:/trusty-amd64-security i386 Packages [437 kB]
Get:7 spacewalk://spacewalk channels:/main amd64 Packages [2,017 kB]
Get:8 spacewalk://spacewalk channels:/main i386 Packages [2,017 kB]
Ign spacewalk://spacewalk channels:/main Translation-en_US
Ign spacewalk://spacewalk channels:/main Translation-en
Ign spacewalk://spacewalk channels:/trusty-amd64-backports Translation-en_US
Ign spacewalk://spacewalk channels:/trusty-amd64-backports Translation-en
Ign spacewalk://spacewalk channels:/trusty-amd64-security Translation-en_US
Ign spacewalk://spacewalk channels:/trusty-amd64-security Translation-en
Ign spacewalk://spacewalk channels:/trusty-amd64-updates Translation-en_US
Ign spacewalk://spacewalk channels:/trusty-amd64-updates Translation-en
Fetched 6,577 kB in 1s (5,320 kB/s)
```
Met many problems, perhaps the repository error.  

Fix Bug, In spacewalk server, change following items:    

```
[root@spacewalk trusty-ia32]# cat Packages | grep -i "Package: python$" -A2
Package: python
Version: 2.7.5-5ubuntu3
Multi-Arch: allowed
[root@spacewalk trusty-ia32]# cat Packages | grep -i "Package: python3$" -A2
Package: python3
Version: 3.4.0-0ubuntu2
Multi-Arch: allowed
[root@spacewalk trusty-ia32]# pwd
/var/cache/rhn/repodata/trusty-ia32
```
And regenerate the Packages.gz via:    

```
# rm -f Packages.gz
# gzip -c Packages Packages.gz
```
Now re-install firefox you won't meet problem.     

### Enable "Push"
Install the packages, precise's package will also be OK for trusty.    

Notice, the Trusty could use saucy based deb.     

```
$ wget https://launchpad.net/~mj-casalogic/+archive/ubuntu/spacewalk-ubuntu/+files/rhncfg_5.10.14-1ubuntu1%7Esaucy2_all.deb
$ wget https://launchpad.net/~mj-casalogic/+archive/ubuntu/spacewalk-ubuntu/+files/osad_5.11.27-1ubuntu1%7Esaucy5_all.deb
$ wget https://launchpad.net/~mj-casalogic/+archive/ubuntu/spacewalk-ubuntu/+files/pyjabber_0.5.0-1.4ubuntu3%7Esaucy1_all.deb
$ sudo dpkg -i osad_5.11.27-1ubuntu1~saucy5_all.deb
$ sudo dpkg -i pyjabber_0.5.0-1.4ubuntu3~saucy1_all.deb
```
Import the certification file and modify the osad's certification file:    

```
# cd /usr/share/rhn
# wget http://spacewalk.example.com/pub/RHN-ORG-TRUSTED-SSL-CERT
# vim /etc/sysconfig/rhn/up2date
sslCACert=/usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT
# vim /etc/sysconfig/rhn/osad.conf
osa_ssl_cert = /usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT
# service osad start
# update-rc.d osad defaults
# reboot
```
If you encounter problem, check the version of the osad/pyjabber, remove the precise
version and install saucy version on Trusty via `dpkg -P osad && dpkg -P pyjabber`,
install the right version, then everything will becomes OK.     

### Enable Remote Command
Install and Configure it via:    

```
# dpkg -i rhncfg_5.10.14-1ubuntu1~saucy2_all.deb 
# rhn-actions-control --enable-run
# mkdir /var/spool/rhn
```
Also add configuration:     
![/images/2015_08_27_15_09_31_375x346.jpg](/images/2015_08_27_15_09_31_375x346.jpg)    

Now you could run command in spacewalk backend.    


### Errata
Following the tips on [http://cefs.steve-meier.de/](http://cefs.steve-meier.de/)    
Import erratas into the system.    

```
# ./errata-import.pl --server spacewalk --errata ./errata.latest.xml  --rhsa-oval \
./com.redhat.rhsa-all.xml --channel centos6-i386 --os-version 6 --publish
```
