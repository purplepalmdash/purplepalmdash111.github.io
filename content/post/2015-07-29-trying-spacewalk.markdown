---
categories: ["Virtualization"]
comments: true
date: 2015-07-29T16:43:23Z
title: Trying SpaceWalk
url: /2015/07/29/trying-spacewalk/
---

### Server Installation && Configration
Server have 2-core and 3072MB, running CentOS6.6, IP address is 10.9.10.2.    

Installation:   

```
# rpm -Uvh http://yum.spacewalkproject.org/2.3/RHEL/6/x86_64/spacewalk-repo-2.3-4.el6.noarch.rpm
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
# yum update
# yum install -y spacewalk-postgresql 
```

Now add a new repository:    

```
# cat /etc/yum.repos.d/jpackage-generic.repo 
[jpackage-generic]
name=JPackage generic
#baseurl=http://mirrors.dotsrc.org/pub/jpackage/5.0/generic/free/
mirrorlist=http://www.jpackage.org/mirrorlist.php?dist=generic&type=free&release=5.0
enabled=1
gpgcheck=1
gpgkey=http://www.jpackage.org/jpackage.asc
```

Met some problems on CentOS6.6, mainly the dependencies problem, switches to CentOS7(10.9.10.100) :    

Install it via:   

```
# rpm -Uvh http://yum.spacewalkproject.org/2.3/RHEL/7/x86_64/spacewalk-repo-2.3-4.el7.noarch.rpm
```

Setup Database for spacewalk:    

```
# yum install spacewalk-setup-postgresql
```

Install spacewalk with postgresql as its backend database:   

```
# yum install spacewalk-postgresql
```
Install spacecmd for using cmd of spacewalker:    

```
# yum install -y spacecmd
```

### Configuration
Configuration, first remove the possible db, then setup.    

```
# /usr/bin/spacewalk-setup-postgresql remove --db
rhnschema --user rhnuser
# spacewalk-setup --disconnected                                          
** Database: Setting up database connection for PostgreSQL backend.
Database "rhnschema" does not exist
** Database: Installing the database:
** Database: This is a long process that is logged in:
** Database:   /var/log/rhn/install_db.log
*** Progress: ##
** Database: Installation complete.
** Database: Populating database.
*** Progress: ############################
* Configuring tomcat.
* Setting up users and groups.
** GPG: Initializing GPG and importing key.
** GPG: Creating /root/.gnupg directory
You must enter an email address.
Admin Email Address? xxxxxx@gmail.com
* Performing initial configuration.
* Activating Spacewalk.
** Loading Spacewalk Certificate.
** Verifying certificate locally.
** Activating Spacewalk.
* Configuring apache SSL virtual host.
Should setup configure apache's default ssl server for you (saves original ssl.conf)
[Y]? y
** /etc/httpd/conf.d/ssl.conf has been backed up to ssl.conf-swsave
* Configuring jabberd.
* Creating SSL certificates.
CA certificate password? 
Re-enter CA certificate password? 
Organization? Lxxzxxx University
Organization Unit [SpaceWalker]? XXU
Email Address [xxxxx@gmail.com]? XXXXX@gmail.com
City? LxxZxx
State? xxxx
Country code (Examples: "US", "JP", "IN", or type "?" to see a list)? CN
** SSL: Generating CA certificate.
** SSL: Deploying CA certificate.
** SSL: Generating server certificate.
** SSL: Storing SSL certificates.
* Deploying configuration files.
* Update configuration in database.
* Setting up Cobbler..
Cobbler requires tftp and xinetd services be turned on for PXE provisioning
functionality. Enable these services [Y]? Y
* Restarting services.
Installation complete.
Visit https://SpaceWalker to create the Spacewalk administrator account.
```

Open the iptables rules:   

```
# iptables -A INPUT -p tcp -m multiport --dport 22,443,5222,69,5432 -j ACCEPT 
```
For better use the spacewalk's utilities, install the following packages:    

```
# yum install -y spacewalk-utils
```
With this package, you could use `spacewalk-common-channels` and other commands.   

### Setup Repository
Add a channel of CentOS7:    

```
[root@SpaceWalker ~]# /usr/bin/spacewalk-common-channels -v -u YourUserName -p \
YourPassword -a x86_64 -k unlimited 'centos7*'
Connecting to http://localhost/rpc/api
Base channel 'CentOS 7 (x86_64)' - creating...
* Activation key 'centos7-x86_64' - creating...
* Child channel 'CentOS 7 Addons (x86_64)' - creating...
** Activation key '1-centos7-x86_64' - adding child channel...
* Child channel 'CentOS 7 Plus (x86_64)' - creating...
** Activation key '1-centos7-x86_64' - adding child channel...
* Child channel 'CentOS 7 Contrib (x86_64)' - creating...
** Activation key '1-centos7-x86_64' - adding child channel...
* Child channel 'CentOS 7 Extras (x86_64)' - creating...
** Activation key '1-centos7-x86_64' - adding child channel...
* Child channel 'CentOS 7 FastTrack (x86_64)' - creating...
** Activation key '1-centos7-x86_64' - adding child channel...
* Child channel 'CentOS 7 Updates (x86_64)' - creating...
** Activation key '1-centos7-x86_64' - adding child channel...
```

Now in the SpaceWalk Backend you will see:    
![/images/2015_07_30_15_22_27_807x526.jpg](/images/2015_07_30_15_22_27_807x526.jpg)    

Import ISO as the repository:    

```
# mkdir /var/distro-trees/centos7_64 -p
# chmod 755 /var/
# chmod 755 /var/distro-trees/
# chmod 755 /var/distro-trees/centos7_64/
# mount -t nfs 192.168.0.79:/home/juju /mnt
#  mount -t iso9660 -o loop /mnt/iso/CentOS-7-x86_64-Everything-1503-01.iso \ 
/var/distro-trees/centos7_64/
# wget https://127.0.0.1/pub/RHN-ORG-TRUSTED-SSL-CERT -O \
/usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT --no-check-certificate 
# /usr/bin/rhnpush -v --channel=centos7-x86_64 --server=https://localhost/APP
--dir="/var/distro-trees/centos7_64/Packages"
Connecting to https://localhost/APP
Username: xxxxxx
Password: 
```
The import process will cost pretty long times.    

### Cobbler Configuration

Via `cobbler check` and fix some errors, notice in CentOS7, the rsyncd is managed via
systemd, so we just enable the rsyncd via:    

```
# systemctl enable rsyncd.service
# systemctl start rsyncd.service
```

debmirror  missing:    

```
# wget http://archive.ubuntu.com/ubuntu/pool/universe/d/debmirror/debmirror_2.10ubuntu1.tar.gz
# tar -xzvf debmirror_2.10ubuntu1.tar.gz
# ...
# cd debmirror-2.10ubuntu1
# make
# cp debmirror /usr/bin/
# cp debmirror.1 /usr/share/man/man1/
# yum install -y cpan
# cpan install Net::INET6Glue 
```

rsync won't be a problem.  

```
# mount -t iso9660 -o loop /media/material/iso/CentOS-7-x86_64-Everything-1503.iso  /mnt1
# cp -ar /mnt1/*  /var/distro-trees/centos7_64
```
Now create the distribution:    

```
# spacecmd  -u YourUserName -p YourPassword -- distribution_create -n centos7 -p /var/distro-trees/centos7_64/ -b centos7-x86_64 -t rhel_6
```

Steps:   

```
# yum install -y spacewalk-postgresql spacewalk-setup-postgresql spacecmd spacewalk-utils firefox
[root@localhost ~]# cat /etc/yum.repos.d/SpaceWalk22.repo 
[SpaceWalk22]
name=SpaceWalk22
baseurl=http://192.168.1.11/SpaceWalk22
enabled=1
gpgcheck=0
```

### Use Answer File
Define the answer file and deploy using answer.txt:    

```
# cat /root/Code/spacewalker/answer.txt
admin-email = root@localhost
ssl-set-org = Spacewalk Org
ssl-set-org-unit = spacewalk
ssl-set-city = My City
ssl-set-state = My State
ssl-set-country = US
ssl-password = spacewalk
ssl-set-email = root@localhost
ssl-config-sslvhost = Y
db-backend=postgresql
db-name=spaceschema
db-user=spacewalk
db-password=spacewalk
db-host=localhost
db-port=5432
enable-tftp=N
```
Reconfigure the LC:    

```
$ sudo su postgres
$ psql
# 
# UPDATE pg_database SET datallowconn = TRUE WHERE datname = 'template0';
# \c template0
# UPDATE pg_database SET datistemplate = FALSE WHERE datname = 'template1';
# drop database template1;
# CREATE DATABASE template1 ENCODING = 'utf8' TEMPLATE = template0 LC_CTYPE =
'en_US.utf8' LC_COLLATE = 'en_US.utf8';
# UPDATE pg_database SET datistemplate = TRUE WHERE datname = 'template1';
# \c template1
# UPDATE pg_database SET datallowconn = FALSE WHERE datname = 'template0';
# \q
```
Now use the answser file for configure the spacewalk installation:    

```
# spacewalk-setup --disconnected --answer-file=/root/Code/spacewalker/answer.txt 
```
Verify the installation, first visit `https://Your_IP`, to setup the username/password.    

Create the channel, 

```
# /usr/bin/spacewalk-common-channels -v -u Username -p Password \ 
-a x86_64 -k unlimited  'centos7*'
# mkdir /var/distro-trees/centos7_64 -p
# chmod 755 /var/
# chmod 755 /var/distro-trees/
# chmod 755 /var/distro-trees/centos7_64/
# cp -ar ContentOfISO /var/distro-trees/centos7_64/
#  spacecmd  -u YourUserName -p YourPass -- distribution_create -n centos7 -p \
/var/distro-trees/centos7_64/ -b centos7-x86_64 -t rhel_7
```

Now check the installed distribution under(System->Kickstart->Distributions):    

![/images/2015_08_03_15_35_04_703x499.jpg](/images/2015_08_03_15_35_04_703x499.jpg)    

Check the kickstart via create the new kickstart file, fail again, why....


### 2.2.1 Version
Use 2.2.1 version:   

```
$ vim jpackage.repo
$ rpm -Uvh http://yum.spacewalkproject.org/2.2/RHEL/6/x86_64/spacewalk-repo-2.2-1.el6.noarch.rpm
$ sudo yum clean all && sudo yum makecache
```


### 2.2.3 Version - Correct way
Disable the cobbler package in epel repository via:    

```
# vim epel.repo 
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=http://mirrors.aliyun.com/epel/7/$basearch
        http://mirrors.aliyuncs.com/epel/7/$basearch
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
### Blacklist cobbler and cobbler-web package ###
exclude=cobbler*
```

Cause the spacewalk has its own cobbler packages in repository, by addeding epel's
cobbler into blacklist we could continue the setup.    
