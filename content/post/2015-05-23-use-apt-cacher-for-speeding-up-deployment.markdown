---
categories: ["linux"]
comments: true
date: 2015-05-23T08:27:55Z
title: Use apt-cacher For Speeding Up Deployment
url: /2015/05/23/use-apt-cacher-for-speeding-up-deployment/
---

### Installation
Install apt-cacher via following command:     

```
$ sudo apt-get install apt-cacher
```

Choose "Daemon" When you see following picture:    

![/images/2015_05_23_08_28_39_418x264.jpg](/images/2015_05_23_08_28_39_418x264.jpg)    

### Configuration
Make sure the configuration `AUTOSTART=1` in `/etc/default/apt-cacher`.    

Enable `allowed_hosts=*` in `/etc/apt-cacher/apt-cacher.conf`.    

Now restart the machine, and check the apt-cacher service via following command:    

```
$ ps -ef | grep apt
www-data   825     1  0 20:34 ?        00:00:00 /usr/bin/perl /usr/sbin/apt-cacher -R 3 -d -p /var/run/apt-cacher.pid
$ sudo netstat -anp | grep 3142
tcp6       0      0 :::3142                 :::*                    LISTEN      825/perl
```

Now when you setup the machines, point the http-proxy into this machine, it will automatically cache the packages.   
