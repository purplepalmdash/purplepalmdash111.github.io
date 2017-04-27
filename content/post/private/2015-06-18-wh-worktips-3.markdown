---
categories: ["virtualization"]
comments: true
date: 2015-06-18T15:50:11Z
title: WH Worktips(3)
url: /2015/06/18/wh-worktips-3/
---

### Prepare Packages
#### System Pacakges
Since CentOS6.5 already deprecated, we need to use `http://vault.centos.org/6.5/` for download the packages.    
Or we could remove all of the repo definition files except the cobbler deployed one.     

We need python-pip for usage. Download it via yum firstly to the directory.     

Download the following packages, these packages may have problems because I downloaded them from the latest repository.    

```
$ yum install yum-plugin-downloadonly
$ yum install --downloadonly --downloaddir=/root/Code/repo/ python-pip
$ yum install --downloadonly --downloaddir=/root/Code/repo/ nethogs
$ yum reinstall --downloadonly --downloaddir=/root/Code/repo/ java7
$ yum reinstall --downloadonly --downloaddir=/root/Code/repo/ mysql-devel
$ yum install --downloadonly --downloaddir=/root/Code/repo/ java-1.7.0-openjdk-devel
$ yum install --downloadonly --downloaddir=/root/Code/repo/ mysql-server
$ yum install --downloadonly --downloaddir=/root/Code/repo/ mysql-devel
[root@z_WHServer repo]# ls
python-pip-1.3.1-4.el6.noarch.rpm
....
```

#### Cloudstack Packages
Download all of the cloudstack packages from `http://cloudstack.apt-get.eu/rhel/4.4/`, and put them into a directory, run `createrepo` for generating the repodata.   

#### PIP Packages
Manually create the offline installation resources.     


### Deployment

```
#  ansible-playbook -i /root/Code/Ansible/hosts --limit=node112 ./cloudstack.yml --tags=base
#  ansible-playbook -i /root/Code/Ansible/hosts --limit=node112 ./cloudstack.yml --tags=mysql
#  ansible-playbook -i /root/Code/Ansible/hosts --limit=node112 ./cloudstack.yml --tags=mysql3306
#  ansible-playbook -i /root/Code/Ansible/hosts --limit=node112 ./cloudstack.yml --tags=csmanagement -vvvv
```
There may be some problems in deployment in `/etc/hosts`, so manually add itself to the hostname:    
!!! node112 !!!

```
[root@node112 yum.repos.d]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.47.58.112    node112
```

From now on, you could visit the management server via:     

[http://10.47.58.112:8080/client/](http://10.47.58.112:8080/client/)    
