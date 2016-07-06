---
categories: ["Virtualization"]
comments: true
date: 2015-10-18T07:34:13Z
title: CloudStackOnUbuntuIssue
url: /2015/10/18/cloudstackonubuntuissue/
---

### 前提
可能我们会碰到这样一种情形，自己的工作机位于某一很复杂的网络环境中，需要在工作
机上搭建出一个网络资源隔离的CloudStack实验环境。而同时我们又希望能最大限度的利
用工作机的性能，尽量让虚拟化中的性能损耗达到最小。那么以下的解决方案可能就是我
们所需要的：物理机上启动一个CloudStack Management的虚拟机，而后在虚拟机的控制
台里将配置好CloudStack Agent的主机加入到CloudStack环境中。而物理机和虚拟机之间
的管理网络为物理机上的一个虚拟网络。这样既做到了硬件资源的最大利用，也有效的杜
绝了本地环境对整个复杂网络环境的影响。    

### 硬件及网络规划
物理机: 4核，支持虚拟化， 内存8G，IP地址为192.168.88.0/24网段里的某台机器。     
虚拟机: 分配2核CPU，内存3G。      
物理机与虚拟机之间，通过虚拟的网络桥接。地址段被规划为172.16.16.0/24，当然你可
以更改为其他网段。    


### 物理机准备
物理机运行Ubuntu 14.04.03 64位版本。    

#### Root登录权限
首先打开root的ssh登录权限:    

```
$ sudo apt-get install -y openssh-server
$ sudo vi /etc/ssh/sshd_config
PermitRootLogin yes
$ sudo bash
# passwd
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
# service ssh restart
```
现在重新登录，即可以使用root直接登录进系统。    

#### 安装软件
安装所需要的软件:    

```
# apt-get update
# apt-get install virt-manager qemu vim wget git nfs-kernel-server
```


#### 创建两个网桥
更改Ubuntu的网络配置文件后，重新启动物理机。这里我们创建了br0和cloudbr0两个网
桥, 注意我们设置cloudbr0的IP地址为172.16.16.1/24。这个网络将作为CloudStack
Management虚拟机与物理机之间的管理网络。       

```
# cat /etc/network/interfaces
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).
    
    # The loopback network interface
    auto lo
    iface lo inet loopback
    
    # The primary network interface
    auto eth0
    iface eth0 inet manual
    
    auto br0
    iface br0 inet dhcp
    bridge_ports eth0
    
    auto cloudbr0
    iface cloudbr0 inet static
    bridge_ports none
    bridge_fd 5
    bridge_stp off
    bridge_maxwait 1
    address 172.16.16.1
    netmask 255.255.255.0
```
创建完毕后，用`ip addr`命令将可以看到5个网络，分别为
lo/eth0/br0/cloudbr0/virbr0， 其中virbr0由libvirtd所创建，这里不用涉及。    

#### 配置cloudbr0的NAT转发
打开内核的转发功能:    

```
# vim /etc/sysctl.conf
net.ipv4.ip_forward=1
```

由于我们的虚拟机需要访问Internet以便安装包，因而我们需要开启cloudbr0上的转发，
配置iptables规则如下:    

```
# iptables -t nat -A POSTROUTING -s \ 
172.16.16.0/24 ! -d 172.16.16.0/24 -j MASQUERADE
```
这样主机端就开启了172.16.16.0/24网段的转发。    

保存我们配置的iptables规则:    

```

# iptables-save >/etc/iptables-save
# echo "pre-up iptables-restore < /etc/iptables-save">>/etc/network/interfaces

```

#### 准备NFS存储
NFS存储可以被用作一级存储或者二级存储,我们这里使用它作为二级存储,主存储即一级
存储我们将使用物理机上的本地存储.    

首先准备存储共享目录:    

```
# mkdir -p /export/primary /export/secondary
```
引出该目录:    

```
# cat >>/etc/exports <<EOM
/export  *(rw,async,no_root_squash,no_subtree_check)
EOM
```
在指定端口配置NFS的statd:    

```
# cp /etc/default/nfs-common /etc/default/nfs-common.orig
# sed -i '/NEED_STATD=/ a NEED_STATD=yes' /etc/default/nfs-common
# sed -i '/STATDOPTS=/ a STATDOPTS="--port 662 \
--outgoing-port 2020"' /etc/default/nfs-common
# diff -du /etc/default/nfs-common.orig /etc/default/nfs-common
```
配置lockd:    

```
# cat >> /etc/modprobe.d/lockd.conf <<EOM
 options lockd nlm_udpport=32769 nlm_tcpport=32803
 EOM
```

重启nfs server:   

```
# service nfs-kernel-server restart
```

#### 网络名配置
配置物理机的网络名如下:   

```
# vim /etc/hostname
physicalnode
# vim /etc/hosts
127.0.0.1       localhost
172.16.16.1     physicalnode
172.16.16.2     cloudstackmgmt

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```

#### 创建CloudStack Management虚拟机
创建一台Ubuntu 14.04 64位虚拟机，特别注意的是其网络设置， 在virt-manager中，手
动选择如下：    

![/images/2015_10_18_08_16_55_654x246.jpg](/images/2015_10_18_08_16_55_654x246.jpg)    

这样虚拟机会选择将自己桥接到物理机上的cloudbr0接口。这时候如果在物理机运行
`brctl show`可以看到如下结果:    

```
# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.52540002d56f       no              eth0
cloudbr0                8000.fe54004d6663       no              vnet0
virbr0          8000.000000000000       yes
```

虚拟机启动完毕后是没有IP地址的，因为cloudbr0上没有提供dhcp服务，我们可以手动配
置IP地址如下, 这里我们把虚拟机的IP地址配置为172.16.16.2:    

```
$ cat /etc/network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet static
address 172.16.16.2
netmask 255.255.255.0
gateway 172.16.16.1
dns-nameservers 223.5.5.5
```
配置完IP地址后，重启后可以验证是否连接到Internet.    

### 虚拟机配置
我们先配置好CloudStack Management虚拟机, 而后再将物理机加入到CloudStack虚拟机
管理网络里.    

#### 开启root登录
步骤和上面物理机上开启root登录一样.    

#### 安装源配置
预先取得用于Ubuntu 14.04 CloudStack环境的安装包, 建立本地源,而后添加为:    

```
# vim /etc/apt/sources.list
.....
deb http://192.168.1.13/iso/    cloudstackdeb/
```

#### 安装包
安装CloudStack Management节点所需的包如下:    

```
# apt-get update
# apt-get install vim wget openntpd cloudstack-management mysql-server
```

注意在安装mysql的时候,需要指定root用户所需的管理密码,这个密码会在后面配置
CloudStack时被用到.    
#### 主机名配置
配置主机名:    

```
# vim /etc/hostname 
CloudStackMgmt
```
配置FQDN所需的主机名:    

```
# vim /etc/hosts
127.0.0.1       localhost
172.16.16.1     physicalnode
172.16.16.2     cloudstackmgmt

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
重启后验证是否被设置正确:    

```
root@cloudstackmgmt:~# hostname
cloudstackmgmt
root@cloudstackmgmt:~# hostname --fqdn
cloudstackmgmt
```

#### 创建数据库
创建CloudStack所需的配置文件,并重启mysql服务:    

```
root@cloudstackmgmt:~# cat /etc/mysql/conf.d/cloudstack.cnf 
    [mysqld]
    innodb_rollback_on_timeout=1
    innodb_lock_wait_timeout=600
    max_connections=350
    log-bin=mysql-bin
    binlog-format = 'ROW'
root@cloudstackmgmt:~# service mysql restart
    mysql stop/waiting
    mysql start/running, process 5812
```
创建CloudStack所需的数据库:    

```
root@cloudstackmgmt:~# cloudstack-setup-databases \
    cloud:engine@localhost \
    --deploy-as=root:xxxxxx \
    -e file -m mymskey44 -k mydbkey00
```
参数说明如下:    

```
# mysql root 密码: xxxxxxx
# cloud user 密码: engine
# management_server_key: mymskey44
# database_key: mydbkey00
```

#### 安装CloudStack系统虚拟机模板
加载NFS共享目录到本地,而后用以下命令安装预先下载好的kvm系统虚拟机所需的模板文
件:    

```
# mount -t nfs 172.16.16.1:/export/secondary /mnt
# /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
-m /mnt -u http://192.168.1.13/iso/systemvm64template-4.5-kvm.qcow2.bz2  -h \
kvm -F
# umount /mnt
```

#### 初始化配置CloudStack
打开浏览器访问[http://172.16.16.2:8080/client](http://172.16.16.2:8080/client)    

如果碰到以下错误, 则看后面的解决步骤:    

![/images/2015_10_18_09_24_35_421x225.jpg](/images/2015_10_18_09_24_35_421x225.jpg)    

关闭tomcat6服务后重启cloudstack-management服务:    

```
# service tomcat6 stop
# service cloudstack-management restart
```
下面的Workaround可以每次重启时自动重启该服务:    

```
# crontab -e
@reboot /bin/RestartCloudStack.sh
# vim /bin/RestartCloudStack.sh
service tomcat6 stop && service cloudstack-management start
```

我们要更改CloudStack使用本地存储,并更改其镜像下载地址:    
本地存储:    

![/images/2015_10_18_09_33_24_669x263.jpg](/images/2015_10_18_09_33_24_669x263.jpg)    

镜像下载地址:    

![/images/2015_10_18_09_35_09_774x233.jpg](/images/2015_10_18_09_35_09_774x233.jpg)    

更改完毕后,`service cloudstack-management restart`重启服务

### 添加CloudStack Agent

#### 物理机端配置
在物理机上,安装和配置CloudStack Agent以及libvirtd, 同样需要配置好本地
CloudStack安装源:    

```
# apt-get install cloudstack-agent
```

配置libvirtd:    

```
$ cp /etc/libvirt/libvirtd.conf /etc/libvirt/libvirtd.conf.orig

$ sed -i '/#listen_tls = 0/ a listen_tls = 0' /etc/libvirt/libvirtd.conf
$ sed -i '/#listen_tcp = 1/ a listen_tcp = 1' /etc/libvirt/libvirtd.conf
$ sed -i '/#tcp_port = "16509"/ a tcp_port = "16509"' /etc/libvirt/libvirtd.conf
$ sed -i '/#auth_tcp = "sasl"/ a auth_tcp = "none"' /etc/libvirt/libvirtd.conf
$ diff -du /etc/libvirt/libvirtd.conf.orig /etc/libvirt/libvirtd.conf
```

Patch libvirt-bin.conf:    

```
$ cp /etc/default/libvirt-bin /etc/default/libvirt-bin.orig
$ sed -i -e 's/libvirtd_opts="-d"/libvirtd_opts="-d -l"/' /etc/default/libvirt-bin
$ diff -du /etc/default/libvirt-bin.orig /etc/default/libvirt-bin
$ service libvirt-bin restart
```

Patch qemu.conf以监听所有端口:    

```
$ cp /etc/libvirt/qemu.conf /etc/libvirt/qemu.conf.orig
$ sed -i '/#vnc_listen = "0.0.0.0"/ a vnc_listen = "0.0.0.0"' /etc/libvirt/qemu.conf
$ diff -du /etc/libvirt/qemu.conf.orig /etc/libvirt/qemu.conf
$ service libvirt-bin restart
```

关闭AppArmor:    

```
$ ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
$ ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
$ apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
$ apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
$ service libvirt-bin restart
```

配置防火墙并打开以下端口:    

```
$ ufw allow proto tcp from any to any port 22
$ ufw allow proto tcp from any to any port 1798
$ ufw allow proto tcp from any to any port 16509
$ ufw allow proto tcp from any to any port 5900:6100
$ ufw allow proto tcp from any to any port 49152:49216
```

#### 配置CloudStack
访问`http://172.16.16.2:8080/client`, 点击`I have used CloudStack before, skip
this guide`.    

Infrastructure -> Zones(View All), 在点开的页面里,点击`Add Zone`.    

选择`Basic`, Next    

![/images/2015_10_18_10_20_49_521x551.jpg](/images/2015_10_18_10_20_49_521x551.jpg)    

选择本地存储,Next:    

![/images/2015_10_18_10_21_48_484x289.jpg](/images/2015_10_18_10_21_48_484x289.jpg)    

跳过配置网络后, 配置Pod IP:    

![/images/2015_10_18_10_23_02_474x419.jpg](/images/2015_10_18_10_23_02_474x419.jpg)    

Guest Traffic配置:    

![/images/2015_10_18_10_24_04_477x337.jpg](/images/2015_10_18_10_24_04_477x337.jpg)    

Cluster名字为:    

![/images/2015_10_18_10_25_05_504x207.jpg](/images/2015_10_18_10_25_05_504x207.jpg)    

添加Host:    

![/images/2015_10_18_10_26_18_469x345.jpg](/images/2015_10_18_10_26_18_469x345.jpg)    

添加二级存储:    

![/images/2015_10_18_10_27_50_501x341.jpg](/images/2015_10_18_10_27_50_501x341.jpg)    

点击`Enable Zone`后可以激活该Zone.    

等待系统虚拟机启动完毕后就可以使用了.    

### 已知问题
CloudStack Agent启动时, 会添加iptables规则,这会造成我们前面加入的转发链失效.      

解决方案: 手动运行命令:     

```
# iptables -t nat -A POSTROUTING -s 172.16.16.0/24 ! -d 172.16.16.0/24 -j \ 
MASQUERADE && iptables -t filter -I FORWARD -j ACCEPT
```

这将使能转发.从而172.16.16.0/24网段的机器能上网.    
