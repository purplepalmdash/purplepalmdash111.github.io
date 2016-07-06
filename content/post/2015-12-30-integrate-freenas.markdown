---
categories: ["Virtualization"]
comments: true
date: 2015-12-30T09:56:50Z
title: Integrate FreeNAS
url: /2015/12/30/integrate-freenas/
---

Use Virtualbox for integrating FreeNAS.    

### Add Disks
Add a new SCSI controller and two disks:    

![/images/2015_12_30_09_58_24_463x329.jpg](/images/2015_12_30_09_58_24_463x329.jpg)   

Then in FreeNAS, import this new disks via:    

![/images/2015_12_30_09_59_19_638x272.jpg](/images/2015_12_30_09_59_19_638x272.jpg)    

### Volume Manager
Add volume of added 2 disks:    

![/images/2015_12_30_10_13_02_811x408.jpg](/images/2015_12_30_10_13_02_811x408.jpg)    

Continue:    

![/images/2015_12_30_10_13_13_589x407.jpg](/images/2015_12_30_10_13_13_589x407.jpg)    

After added:   

![/images/2015_12_30_10_14_56_1014x232.jpg](/images/2015_12_30_10_14_56_1014x232.jpg)   

### ISCSI Sharing
Create new sharing:    

![/images/2015_12_30_10_19_26_499x354.jpg](/images/2015_12_30_10_19_26_499x354.jpg)    

Add portal:    

![/images/2015_12_30_10_20_53_417x362.jpg](/images/2015_12_30_10_20_53_417x362.jpg)    

Add Initiator:   

![/images/2015_12_30_10_22_11_456x202.jpg](/images/2015_12_30_10_22_11_456x202.jpg)   

Add Target:    

![/images/2015_12_30_10_25_57_574x436.jpg](/images/2015_12_30_10_25_57_574x436.jpg)   

Add Extent:    

![/images/2015_12_30_10_31_42_489x572.jpg](/images/2015_12_30_10_31_42_489x572.jpg)   

Extend options:    

![/images/2015_12_30_10_31_18_462x357.jpg](/images/2015_12_30_10_31_18_462x357.jpg)    

Associate:    

![/images/2015_12_30_10_33_49_663x363.jpg](/images/2015_12_30_10_33_49_663x363.jpg)   

Enable the iscsi service:    

![/images/2015_12_30_10_34_55_333x331.jpg](/images/2015_12_30_10_34_55_333x331.jpg)   

### Integration
Integaration with cloudstack would be looked like following:    

![/images/2015_12_30_11_49_54_375x423.jpg](/images/2015_12_30_11_49_54_375x423.jpg)     
