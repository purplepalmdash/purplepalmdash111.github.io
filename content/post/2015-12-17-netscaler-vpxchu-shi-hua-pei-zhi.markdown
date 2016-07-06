---
categories: ["Virtualization"]
comments: true
date: 2015-12-17T14:08:35Z
title: NetScaler VPX初始化配置
url: /2015/12/17/netscaler-vpxchu-shi-hua-pei-zhi/
---

### 初始化配置
启动虚拟机以后，通过nsroot/nsroot登录入VPX.   

清除所有配置:    

![/images/2015_12_17_14_20_54_580x93.jpg](/images/2015_12_17_14_20_54_580x93.jpg)   

如下，做IP配置:    

![/images/2015_12_17_14_23_29_728x153.jpg](/images/2015_12_17_14_23_29_728x153.jpg)   

初始化配置完毕以后，即可在web后台进行配置。    

### License
申请license的时候注意，选择的MAC地址不能有任何的`:`符号， 例如52:54:00:这种就不能通过成
功。 在Netscaler上可以通过以下命令查看host id:    

```
root@ns# lmutil lmhostid 
lmutil - Copyright (c) 1989-2013 Flexera Software LLC. All Rights Reserved. 
The FlexNet host ID of this machine is "xxxxxxx"
```
查看激活后的license情形:   

```
> sh license
        License status:
                           Web Logging: YES
                      Surge Protection: YES
                        Load Balancing: YES
                     Content Switching: YES
....
```
参考:     
[http://sam.yeung.blog.163.com/blog/static/222663482013811102013782/](http://sam.yeung.blog.163.com/blog/static/222663482013811102013782/)    
