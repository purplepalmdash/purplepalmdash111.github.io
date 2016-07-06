---
categories: ["Embedded"]
comments: true
date: 2014-10-18T00:00:00Z
title: Embedded Environment on Arch
url: /2014/10/18/embedded-environment-on-arch/
---

### NFS Server
Server Setup

```
$ sudo pacman -S nfs-utils
$ sudo vim /etc/idmapd.conf
[General]

#Verbosity = 0
Verbosity = 1
Pipefs-Directory = /var/lib/nfs/rpc_pipefs
Domain = localdomain

[Mapping]

Nobody-User = nobody
Nobody-Group = nobody

$ sudo vim /etc/conf.d/nfs-common.conf
STATD_OPTS="-p 32765 -o 32766 -T 32803"

$ sudo vim  /etc/conf.d/nfs-server.conf
MOUNTD_OPTS="-p 20048"

$ sudo mkdir -p /srv/nfs4/music
$ cat /etc/exports
/srv/nfs4/ 10.0.0.0/24(rw,fsid=root,no_subtree_check)
/srv/nfs4/music 10.0.0.0/24(rw,no_subtree_check,nohide) # note the nohide option which is applied to mounted directories on the file system.
$ sudo exportfs -rav
$ sudo systemctl restart nfs-server.service

```
Testing Server:    

```
$ sudo mount -t nfs 10.0.0.221:/srv/nfs4 /mnt
$ ls /mnt

```
### tftpd Server
Server:    

```
$ sudo pacman -S tftp-hpa
$ sudo vim /etc/systemd/system/tftpd.service
[Unit]
Description=hpa's original TFTP daemon

[Service]
ExecStart=/usr/sbin/in.tftpd -s /srv/tftp/
StandardInput=socket
StandardOutput=inherit
StandardError=journal
$ sudo systemctl start tftpd.socket
$ sudo systemctl enable tftpd.socket

```
Client(10.0.0.230):    

```
$ tftp 10.0.0.221
tftp> get abc.txt
tftp> quit
$ ls -l abc.txt
-rw-r--r-- 1 root root 0 Oct 18 20:23 abc.txt

```

### NFS Servers(Easy)

```
$ sudo pacman -S nfs-utils
$ cat /etc/exports
/media/home/xxxx *(rw,sync,no_subtree_check)
$ sudo systemctl enable rpcbind.service
$ sudo systemctl start rpcbind.service
$ sudo systemctl enable nfs-server.service
$ sudo systemctl start nfs-server.service
```
