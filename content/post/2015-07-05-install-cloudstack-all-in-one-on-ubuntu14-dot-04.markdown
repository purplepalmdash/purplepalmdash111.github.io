---
categories: ["Virtualization"]
comments: true
date: 2015-07-05T16:08:59Z
title: Install CloudStack All-In-One On Ubuntu14.04
url: /2015/07/05/install-cloudstack-all-in-one-on-ubuntu14-dot-04/
---

Refers to:     
[http://www.greenhills.co.uk/2015/02/23/cloudstack-4.4-single-server-on-ubuntu-14.04.1-with-kvm.html](http://www.greenhills.co.uk/2015/02/23/cloudstack-4.4-single-server-on-ubuntu-14.04.1-with-kvm.html)   


### Preparation For Packages
Install reprepro:    

```
# apt-get install -y reprepro
```

```
# mkdir -p /srv/reprepro
root@DebServer:/srv/reprepro# ls
conf  db  dists  incoming  indices  lists  logs  pool  project  tmp
root@DebServer:/srv/reprepro# ls conf/
distributions  incomming  updates  uploaders
root@DebServer:/srv/reprepro# cat conf/distributions 
Origin: Alveonet
Label: Alveonet
Suite: trusty
Codename: trusty
Version: 14.04
Architectures: i386 amd64 source
Components: main cloudstack43 cloudstack44 cloudstack45 cloudstack
Description: Alveonet specific (or backported) packages
#SignWith: dash1982
DebOverride: ../indices/override.trusty.main
UDebOverride: ../indices/override.tursy.main.debian-installer
DscOverride: ../indices/override.trusty.main.src
DebIndices: Packages Release . .gz .bz2
UDebIndices: Packages . .gz .bz2
DscIndices: Sources Release .gz .bz2
Contents: . .gz .bz2
Update: - cloudstack45 cloudstack43 cloudstack44
Log: packages.alveonet.org.log
root@DebServer:/srv/reprepro# cat conf/incomming 
Name: default
IncomingDir: incoming
TempDir: tmp
Allow: trusty trusty-backports
Cleanup: on_deny on_error
root@DebServer:/srv/reprepro# cat conf/updates 
Name: cloudstack44
Method: http://cloudstack.apt-get.eu/ubuntu
#VerifyRelease: 86C278E3
Suite: trusty
Architectures: amd64
GetInRelease: no
#Components: 4.3>main
Components: 4.4>cloudstack44

# Name: cloudstack44
# Method: http://cloudstack.apt-get.eu/ubuntu
# #VerifyRelease: 86C278E3
# Suite: trusty
# Architectures: amd64
# GetInRelease: no
# #Components: 4.3>main
# Components: 4.4>main


Name: cloudstack45
Method: http://cloudstack.apt-get.eu/ubuntu
#VerifyRelease: 86C278E3
Suite: trusty
Architectures: amd64
GetInRelease: no
#Components: 4.3>main
Components: 4.5>cloudstack45


Name: cloudstack43
Method: http://cloudstack.apt-get.eu/ubuntu
#VerifyRelease: 86C278E3
Suite: trusty
Architectures: amd64
GetInRelease: no
#Components: 4.3>main
Components: 4.3>cloudstack43
root@DebServer:/srv/reprepro# cat conf/uploaders 
allow * by unsigned



# reprepro -Vb /srv/reprepro export
# reprepro update

```

By doing this you could make a local reprepro created repository.     

TBD

### Installation Steps

#### System Preparation
First install Ubuntu14.04, I use Cobbler Server's PXE Installation.    

Change the Root Permit login under `/etc/ssh/sshd_config` and restart the ssh service.    

Change the IP Address to 10.10.10.2 via:    

```
root@Ubuntu-14:~# cat /etc/network/interfaces
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet static
address 10.10.10.2
netmask 255.255.255.0
gateway 10.10.10.1
dns-nameservers 180.76.76.76
```

Relogin to the system, and begin to install the packages.  

```
# vi /etc/apt/source.list
deb http://192.168.1.111 amd64/
deb http://192.168.1.111/reprepro/      trusty  cloudstack44
# apt-get update && apt-get install vim
```

I use 192.168.1.111 because I cached all of the required packages in this server, thus it could speed-up the deployment.     

#### Server Configuration
Configure the Server's hostname:    

```
root@Ubuntu-14:~# cat /etc/hostname 
CS
root@Ubuntu-14:~# cat /etc/hosts
127.0.0.1       localhost
10.10.0.2       CS
127.0.1.1       CS.CSDomain     CS

root@Ubuntu-14:~# reboot
# hostname --fqdn
CS
```
Now configure the NTP via:   

```
# apt-get purge ntp
# apt-get install openntpd
```

Configure the Network:    

```
# apt-get install bridge-utils
# cat /etc/network/interfaces
    
    # The loopback network interface
    auto lo
    iface lo inet loopback
    
    # The primary network interface
    auto eth0
    iface eth0 inet manual
    
    
    auto cloudbr0
    iface cloudbr0 inet static
            address 10.10.10.2
            netmask 255.255.255.0
            gateway 10.10.10.1
            dns-nameservers 180.76.76.76
            bridge-ports eth0
            bridge_fd 5
            bridge_stp off
            bridge_maxwait 1
    
    # Private network
    auto cloudbr1
    iface cloudbr1 inet manual
            bridge_ports none
            bridge_fd 5
            bridge_stp off
            bridge_maxwait 1
# reboot
```
Now the Network Bridge is configured.    

#### CloudStack Installation
Since all of the packages are cached into LAN environment, simply type following command for installing:    

```
# apt-get install cloudstack-management
```

Then Install the database server:    

```
# apt-get install mysql-server
# vim /etc/mysql/conf.d/cloudstack.cnf
[mysqld]
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=350
log-bin=mysql-bin
binlog-format = 'ROW'
root@CS:~# service mysql restart
root@CS:~# cloudstack-setup-databases cloud:engine@localhost --deploy-as=root:engine -e file -m mymskey44 -k mydbkey00
```

#### NFS Storage Preparation
Add export directory:   

```
# mkdir -p /export/primary /export/secondary
# apt-get install nfs-kernel-server
# vim /etc/exports
/export  *(rw,async,no_root_squash,no_subtree_check)
# cp /etc/default/nfs-common /etc/default/nfs-common.orig
# sed -i '/NEED_STATD=/ a NEED_STATD=yes' /etc/default/nfs-common
# sed -i '/STATDOPTS=/ a STATDOPTS="--port 662 --outgoing-port 2020"' /etc/default/nfs-common
# diff -du /etc/default/nfs-common.orig /etc/default/nfs-common

root@CS:~# cat /etc/modprobe.d/lockd.conf 
options lockd nlm_udpport=32769 nlm_tcpport=32803
root@CS:~# service nfs-kernel-server restart
root@CS:~# showmount -e 127.0.0.1
Export list for 127.0.0.1:
/export *
```

Mount it via:    

```
root@CS:~# mkdir -p /mnt/primary /mnt/secondary
root@CS:~# tail -2 /etc/fstab
10.10.10.2:/export/primary   /mnt/primary    nfs rsize=8192,wsize=8192,timeo=14,intr,vers=3,noauto  0   2
10.10.10.2:/export/secondary /mnt/secondary  nfs rsize=8192,wsize=8192,timeo=14,intr,vers=3,noauto  0   2
root@CS:~# mount /mnt/primary/
root@CS:~# mount /mnt/secondary/
root@CS:~# mount | tail -2
10.10.10.2:/export/primary on /mnt/primary type nfs (rw,rsize=8192,wsize=8192,timeo=14,intr,vers=3,addr=10.10.10.2)
10.10.10.2:/export/secondary on /mnt/secondary type nfs (rw,rsize=8192,wsize=8192,timeo=14,intr,vers=3,addr=10.10.10.2)
```

#### Cloudstack Agent Installation
Install it via:    

```
root@CS:~# apt-get install cloudstack-agent
root@CS:~# cp /etc/libvirt/libvirtd.conf /etc/libvirt/libvirtd.conf.orig
root@CS:~# sed -i '/#listen_tls = 0/ a listen_tls = 0' /etc/libvirt/libvirtd.conf
root@CS:~# sed -i '/#listen_tcp = 1/ a listen_tcp = 1' /etc/libvirt/libvirtd.conf
root@CS:~# sed -i '/#tcp_port = "16509"/ a tcp_port = "16509"' /etc/libvirt/libvirtd.conf
root@CS:~# sed -i '/#auth_tcp = "sasl"/ a auth_tcp = "none"' /etc/libvirt/libvirtd.conf
root@CS:~# diff -du /etc/libvirt/libvirtd.conf.orig /etc/libvirt/libvirtd.conf
root@CS:~# cp /etc/default/libvirt-bin /etc/default/libvirt-bin.orig
root@CS:~# sed -i -e 's/libvirtd_opts="-d"/libvirtd_opts="-d -l"/' /etc/default/libvirt-bin
root@CS:~# service libvirt-bin restart
root@CS:~# cp /etc/libvirt/qemu.conf /etc/libvirt/qemu.conf.orig
root@CS:~# sed -i '/#vnc_listen = "0.0.0.0"/ a vnc_listen = "0.0.0.0"' /etc/libvirt/qemu.conf
root@CS:~# diff -du /etc/libvirt/qemu.conf.orig /etc/libvirt/qemu.conf
root@CS:~# service libvirt-bin restart
root@CS:~# ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
root@CS:~# ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
root@CS:~# apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
root@CS:~# apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
root@CS:~# service libvirt-bin restart
```

#### Configure FireWall
Open the following ports:    

```
root@CS:~# ufw allow proto tcp from any to any port 22
Rules updated
Rules updated (v6)
root@CS:~# ufw allow proto tcp from any to any port 1798
Rules updated
Rules updated (v6)
root@CS:~# ufw allow proto tcp from any to any port 16509
Rules updated
Rules updated (v6)
root@CS:~# ufw allow proto tcp from any to any port 5900:6100
Rules updated
Rules updated (v6)
root@CS:~# ufw allow proto tcp from any to any port 49152:49216
Rules updated
Rules updated (v6)
```

Now restart and verify the NFS working.    

```
root@CS:~# rpcinfo -u 10.10.10.2 mount
program 100005 version 1 ready and waiting
program 100005 version 2 ready and waiting
program 100005 version 3 ready and waiting
root@CS:~# showmount -e 10.10.10.2
Export list for 10.10.10.2:
/export *
root@CS:~# mount /mnt/primary/
root@CS:~# mount /mnt/secondary/
```

Added the System Template:    

```
root@CS:~# /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt -m /mnt/secondary -u http://192.168.1.111/systemvm64template-4.4.1-7-kvm.qcow2.bz2  -h kvm -F
......
Successfully installed system VM template  to /mnt/secondary/template/tmpl/1/3/
```

### Cloudstack
First manually start the services:     

```
service cloudstack-management status
service cloudstack-agent status
service tomcat6 status

service cloudstack-management stop
service tomcat6 stop
service cloudstack-agent stop
ps -efl | grep java

service cloudstack-management start
service cloudstack-management status
service cloudstack-agent start
service cloudstack-agent status
```

Now setup the UI via visiting `http://10.10.10.2:8080/client`, usename/password: admin/password:    

```
Choose "Continue with basic installation". This will start a wizard that will walk through the configuration. You will need to adjust the network values for your environment, and make sure you use appropiate, free, ranges.

Add a new zone named "zone1", DNS1 10.10.10.1 and Internal DNS 10.10.10.1.
Add a new pod named "pod1", gateway 10.10.10.1, netmask 255.255.255.0, IP range 10.10.10.160-10.10.10.169.
Add a guest network, gateway 10.10.10.1, netmask 255.255.255.0, IP range 10.10.10.170-192.168.77.230.
Add a cluster named cluster1, Hypervisor KVM.
Add a host. Host Name "CS", user root, passsword for the root linux user.
Add primary storage: name primary1, protocol NFS, Scope Cluster, server 10.10.10.10, path /export/primary.
Add secondary storage: NFS server 10.10.10.10, path /export/secondary.
Hit Launch and pray.
This should go through a sequence of setup.
```

Now you could enjoy the cloudstack all-in-one
