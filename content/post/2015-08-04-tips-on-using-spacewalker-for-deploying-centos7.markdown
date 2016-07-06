---
categories: ["Virtualization"]
comments: true
date: 2015-08-04T15:57:47Z
title: Tips On Using SpaceWalker For Deploying CentOS7
url: /2015/08/04/tips-on-using-spacewalker-for-deploying-centos7/
---

### Configuration
After SpaceWalker has been setup, the configuration we need to done is listed as
following:    

```
In /etc/rhn/rhn.conf change the value of the parameter cobbler.host to the ip address of the spacewalk server.
In /etc/cobbler/settings change the value of the parameters server and redhat_management_server to the ip-address of the spacewalk server.
```
Install cobbler bootloaders via:    

```
# yum install -y cobbler-loaders
```

### Build Customized ISO
via `cobbler buildiso`, and in the same folder you will get generated.iso.    

### Add Customized Channel
Under the Channel-> Management Software Channel-> Create New Channel.   

![/images/2015_08_05_16_50_50_649x442.jpg](/images/2015_08_05_16_50_50_649x442.jpg)   

Repo sync via:    

```
# spacewalk-repo-sync -c spacewalker_rhel7_23 -u \ 
http://yum.spacewalkproject.org/2.3/RHEL/7/x86_64/
```

### Add Ubuntu Deployment
