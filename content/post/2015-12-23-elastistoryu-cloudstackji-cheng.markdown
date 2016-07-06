---
categories: ["Virtualization"]
comments: true
date: 2015-12-23T17:25:55Z
title: ElastiStor与CloudStack集成
url: /2015/12/23/elastistoryu-cloudstackji-cheng/
---

### 安装ElastiStor
建立虚拟机，4G内存， 2核，插入光盘后，进入到以下界面:    

![/images/2015_12_23_17_25_15_759x415.jpg](/images/2015_12_23_17_25_15_759x415.jpg)   

选择`both Center and Node`, 进入到以下界面后，配置网络相关信息:    

![/images/2015_12_23_17_28_36_720x396.jpg](/images/2015_12_23_17_28_36_720x396.jpg)    

search domain这一项配置如下， 随便添加某个域名，例如www.baidu.com:    

安装过程:    

![/images/2015_12_23_17_33_29_644x327.jpg](/images/2015_12_23_17_33_29_644x327.jpg)    

选择时区：    

![/images/2015_12_23_17_36_34_712x398.jpg](/images/2015_12_23_17_36_34_712x398.jpg)    

安装完毕后，进入到登录界面:    

![/images/2015_12_23_17_38_40_539x422.jpg](/images/2015_12_23_17_38_40_539x422.jpg)  

默认密码就是我们在前面设置的，默认用户名是root.    

### 登录
首次登录会遇到问题的解决:    

![/images/2015_12_23_18_11_05_844x186.jpg](/images/2015_12_23_18_11_05_844x186.jpg)   

以及:    

![/images/2015_12_23_18_12_25_866x213.jpg](/images/2015_12_23_18_12_25_866x213.jpg)   


![images/2015_12_23_18_15_43_647x176.jpg](/images/2015_12_23_18_15_43_647x176.jpg)    

现在可以访问登录界面, 默认用户名/密码为admin/password.        

[/images/2015_12_23_18_15_49_678x413.jpg](/images/2015_12_23_18_15_49_678x413.jpg)    

### 配置

后台可以可视化来配置， 从site-> HA Group-> Node， 添加完毕后我们可以看到以下界面：   

![/images/2015_12_23_19_32_13_819x326.jpg](/images/2015_12_23_19_32_13_819x326.jpg)



