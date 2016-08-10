+++
categories = ["Virtualization"]
date = "2016-08-10T14:29:27+08:00"
description = "Setup CloudStack Site to Site VPN"
keywords = ["CloudStack"]
title = "CloudStackSiteToSiteVPNHowTo"

+++
Recently I am investigating the VPN solution on company's private cloud. Some guys
insist on setting up the site-to-site VPN on Juniper's firewall(SR240), but I think
this is a piece of shit. Setting up the VPN on physical firewall will lose the `ease of
use` and `ease of configuration` of CloudStack's built-in VPN functionality, so
following are the steps for me to investigate the CloudStack's site-to-site VPN among
different vpcs. Follow the guide line you will get two or more geographical independent
sites working as in a big ethernet.    

### Prerequisites
The environment is pretty simple, you need 2 PCs(Or more) for experiment.    

![/images/2016_08_10_14_45_50_603x334.jpg](/images/2016_08_10_14_45_50_603x334.jpg)    

Working Machine 1's configuration:    

```
Intel(R) Core(TM) i7-3770 CPU @ 3.40GHz
24 G Memory
1000M Ethernet Card
More than 20G Free disk    
ArchLinux
```

Working Machine 2's configuration:    

```
Intel(R) Core(TM) i3 CPU         540  @ 3.07GHz
8 G Memory
1000M Ethernet Card
More than 20G Free disk    
ArchLinux
```
Make sure the nested KVM is configurated in both machine:    

```
$ systool -m kvm_intel -v | grep nested
    nested              = "Y"
```
Also make sure you have a Linux Bridge(Not the physical card or OpenVswitch Bridge!).   

### Testing VM
We create two virtual machines for installing cloudstack all in one environment, like
following picture.   

![/images/2016_08_10_14_53_23_652x427.jpg](/images/2016_08_10_14_53_23_652x427.jpg)     

Virtual Machine 1:    

```
4 core(host-passthrough CPU)
8G Memory
200G Disk
Bridged to br0
CentOS 6.8 X86_64
```

Virtual Machine 2(Because working Machine 2 only have 8G memory, we allocated 7G to
it):        

```
4 core(host-passthrough CPU)
7G Memory
200G Disk
Bridged to br0
CentOS 6.8 X86_64
```

The network configuration is listed as:    

![/images/2016_08_10_14_56_33_689x188.jpg](/images/2016_08_10_14_56_33_689x188.jpg)    

Specified `shared` is very important, or you won't get networking works.    

