+++
title = "WorkingTipsOnRPMBuild"
date = "2017-06-21T11:33:40+08:00"
categories = ["Linux"]
keywords = ["Linux"]
description = "WorkingTipsOnRPMBuild"

+++
### Prerequisites
From docker images Centos:6.6.    

### Steps
#### Environment Preparation
Start the docker instance:    

```
$ sudo docker run -it centos:6.6 /bin/bash
```
Install the dev environment(C):  

```
# yum install -y rpm-build rpmdevtools vim gcc tar openssh-clients

```
Create the macro for rpmbuild, and setup the rpm build tree:   

```
# vim /root/.rpmmacros
%_topdir    /root/rpmbuild
# rpmdev-setuptree
```

#### C Project
Refers to:    

[https://blog.packagecloud.io/rpm/rpmbuild/packaging/2015/06/29/building-rpm-packages-with-rpmbuild/](https://blog.packagecloud.io/rpm/rpmbuild/packaging/2015/06/29/building-rpm-packages-with-rpmbuild/)    

#### Verification
Using a new docker instance, then you could verify your rpm installation and
uninstallation.    
