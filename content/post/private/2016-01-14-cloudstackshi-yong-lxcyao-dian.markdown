---
categories: ["Virtualization"]
comments: true
date: 2016-01-14T19:41:03Z
title: CloudStack使用LXC要点
url: /2016/01/14/cloudstackshi-yong-lxcyao-dian/
---

### 先决条件
Cloudstack 4.6.0, Management Server搭建完毕, Agent包安装完毕.     
具体的步骤可以参考KVM的安装.    

### Agent机器
设置hypervisor的类型为LXC模式:    

```
$ vim /etc/cloudstack/agent/agent.properties
 hypervisor.type=lxc
$ service cloudstack-agent restart
```

### LXC系统虚拟机模板安装
在CloudStack Management节点上,运行以下命令安装系统虚拟机模板:      

```
$ /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt -m /mnt1 -u http://192.168.177.13/systemvm64template-4.6.0-kvm.qcow2.bz2 -h lxc -F
```
注意替换以上命令里的镜像下载地址及安装目录.    

### 添加Zone
先添加一个基本的Zone,注意选择hypervisor为LXC.    
添加一级/二级存储.    
添加完毕后, enable zone后, 下载模板, 注意不要选择HVM的选项.   


到这里就可以开始创建实例了.      
