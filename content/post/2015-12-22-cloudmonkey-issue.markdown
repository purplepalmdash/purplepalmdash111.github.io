---
categories: ["virtualization"]
comments: true
date: 2015-12-22T21:18:21Z
title: CloudMonkey Issue
url: /2015/12/22/cloudmonkey-issue/
---

### Could not start cloudmonkey

After installing cloudstack, cloudmonkey couldnot be used, the reason is
listed as following:     

```
% cloudmonkey
Traceback (most recent call last):
  File "/usr/local/bin/cloudmonkey", line 5, in <module>
    from pkg_resources import load_entry_point
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/pkg_resources.py", line 2603, in <module>
    working_set.require(__requires__)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/pkg_resources.py", line 666, in require
    needed = self.resolve(parse_requirements(requirements))
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/pkg_resources.py", line 565, in resolve
    raise DistributionNotFound(req)  # XXX put more info here
pkg_resources.DistributionNotFound: requests
```

The solution is via:    

```
# easy_install --upgrade pip
# pip install --upgrade setuptools
# pip install --upgrade distribute
# wget https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py -O - | python
```

So Now you could use cloudstack.    

### Use Ansible Together With Cloudmonkey

I think because my cloudstack agent didn't use a bridge, so the cloudmonkey runs failed.   

### Cloudmonkey tips
Configure the secstorage download address for local usage.   

```
# cloudmonkey update configuration name=secstorage.allowed.internal.sites
 value=192.168.0.0/16
```

Create a new service offering:    

```
# cloudmonkey create serviceoffering cpunumber=1 cpuspeed=500 displaytext=monkeyking
memory=256 name=monkeyking
```

Register the new template:    

```
cloudmonkey register template hypervisor=kvm zoneid=2ee04a60-499c-bcc6-c3ba92624b35 format=qcow2 name=tinycore displaytext=tinycoretemplate ispublic=true ostypeid=11 url=http://192.168.0.79/tiny.qcow2
```

The ostypeid could be adjusted in the graphical interface, while the ostypeid for
`other Linux(64-bit)` is    

```
ostypeid = ff2564a4-a91e-11e5-9855-52540048638c
```

Start an instance via cloudmonkey, refers to following command:   

```
$ Deploy VirtualMachine ZoneID=<zoneid> ServiceOfferingID=<serviceofferingid> TemplateID=<templateid> StartVM=false IPAddress=192.168.30.10 Name=My-VirtualMachine DisplayName=“My Virtual Machine”
```
