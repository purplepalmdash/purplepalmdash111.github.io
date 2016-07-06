---
categories: ["Virtualization"]
comments: true
date: 2016-01-31T11:49:13Z
title: virtio-gpu working tips
url: /2016/01/31/virtio-gpu-working-tips/
---

### System
Install Fedora 23, select fedora server and install the system, then using the
lastest development kernel via:    

```
$ curl -s https://repos.fedorapeople.org/repos/thl/kernel-vanilla.repo | sudo tee /etc/yum.repos.d/kernel-vanilla.repo
$ sudo dnf --enablerepo=kernel-vanilla-mainline update
```

Checking the running kernel via:    

```
$ uname -r
4.5.0-0.rc1.git0.1.vanilla.knurd.1.fc23.x86_64
```

Running the system which have kernel version newer than 4.4  is the basis for enable the virt-io.   

### Install packages

```
$ sudo dnf install -y gcc zlib-devel glib2-devel pixman-devel libfdt-devel \ 
lzo-devel snappy-devel bzip2-devel libseccomp-devel gtk2-devel gtk3-devel \ 
gnutls-devel vte-devel SDL-devel librdmacm-devel libuuid-devel \ 
 libcap-ng-devel libcurl-devel ceph-devel libssh2-devel libaio-devel \ 
glusterfs-devel glusterfs-api-devel numactl-devel gperftools-devel \ 
 texinfo libiscsi-devel spice-server-devel libusb-devel usbredir-devel \ 
libnfs-devel libcap-devel libattr-devel  
```

### virglrenderer
coprs have the repository for this:    
[https://copr.fedorainfracloud.org/coprs/kraxel/](https://copr.fedorainfracloud.org/coprs/kraxel/)    

```
$ cd /etc/yum.repos.d/
$ sudo wget https://copr.fedorainfracloud.org/coprs/kraxel/virgl/repo/fedora-23/kraxel-virgl-fedora-23.repo
```

Install the virglrenderer via:    

```
$ sudo dnf install virglrenderer
```

### qemu
Git clone the source code:    

```
$ git clone https://www.kraxel.org/cgit/qemu
```

