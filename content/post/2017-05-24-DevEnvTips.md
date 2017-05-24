+++
date = "2017-05-24T15:28:51+08:00"
keywords = ["Virtualization"]
description = "Dev envs for lab"
title = "DevEnvsForLAB"
categories = ["Virtualization"]

+++
### 目的
LAB验证环境的搭建。    
前提条件： 最小化CentOS7系统迁移（见前一篇文章）.    

### 安装
CentOS 7.3(1611), 最小化安装。    

```
# yum update -y
# yum install -y vim qemu libvirt libvirt-devel ruby-devel gcc qemu-kvm
net-tools virt-manager wget
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# yum install -y nethogs byobu ansible rubygem-ruby-libvirt.x86_64
# wget https://releases.hashicorp.com/vagrant/1.9.1/vagrant_1.9.1_x86_64.rpm
# yum install vagrant_1.9.1_x86_64.rpm
# vagrant plugin install --plugin-version  0.0.37 vagrant-libvirt
# cd ~/.vagrant.d/gems/2.2.5/gems
# ln -s ../extensions ./
# vi /etc/modprobe.d/kvm-nested.conf
options kvm_intel nested=1
```
Disable the selinux, firewalld:    

```
# vim /etc/selinux/config
SELINUX=disabled
# systemctl disable firewalld
```
Now restart the machine, your dev environment is ready now.    
