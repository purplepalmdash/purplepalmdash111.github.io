---
categories: ["linux"]
comments: true
date: 2015-05-19T11:57:19Z
title: Setup Local Repository
url: /2015/05/19/setup-local-repository/
---

### Ubuntu 
After using apt-mirror syncing all of the packages from the repository website, setup a ftp site:    

```
# apt-get install -y proftpd
# cat conf.d/anonymous.conf 
<Anonymous ~ftp>
   User                    ftp
   Group                nogroup
   UserAlias         anonymous ftp
   RequireValidShell        off
#   MaxClients                   10
   <Directory *>
     <Limit WRITE>
       DenyAll
     </Limit>
   </Directory>
 </Anonymous>
#  mount --bind /mnt/myrepo/mirror/mirrors.aliyun.com/ /srv/ftp/
# service proftpd restart
```

Now Open your browser to `ftp://Your_URL/`, you will find the repository available.    

### CentOS Proftpd
Just remember the default directory is located at `/var/ftp`, 

```
# yum install -y proftpd
# mount --bind /mirror/mirrors.aliyun.com/ /var/ftp/
# service proftpd restart
```

### Client Configuration

Replace the URL into your ftp url:    


```
# vim /etc/apt/sources.list
deb ftp://YourURL/ubuntu/ trusty main restricted universe multiverse
deb ftp://YourURL/ubuntu/ trusty-security main restricted universe multiverse
deb ftp://YourURL/ubuntu/ trusty-updates main restricted universe multiverse
deb ftp://YourURL/ubuntu/ trusty-proposed main restricted universe multiverse
deb ftp://YourURL/ubuntu/ trusty-backports main restricted universe multiverse
deb-src ftp://YourURL/ubuntu/ trusty main restricted universe multiverse
deb-src ftp://YourURL/ubuntu/ trusty-security main restricted universe multiverse
deb-src ftp://YourURL/ubuntu/ trusty-updates main restricted universe multiverse
deb-src ftp://YourURL/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src ftp://YourURL/ubuntu/ trusty-backports main restricted universe multiverse
# apt-get update && apt-get upgrade 
```

Using local repository will greately improve your development speed.    


### Use apache way(On CentOS6.6)    
By using the symlinks and enable the httpd to start automatically at systemboot.    

```
[root:/var/www]# cd html/
[root:/var/www/html]# ln /home/juju/myrepo/mirror/mirrors.aliyun.com/ubuntu -s
[root:/var/www/html]# chkconfig httpd on
```


