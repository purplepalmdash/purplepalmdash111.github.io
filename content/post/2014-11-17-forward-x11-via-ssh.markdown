---
categories: ["linux"]
comments: true
date: 2014-11-17T00:00:00Z
title: Forward x11 via ssh
url: /2014/11/17/forward-x11-via-ssh/
---

Since the 5901 port is forbiddended via administrator of the switch, we have to forward the traffic to remote machine via ssh:    

First in our machine type following command:    

```
ssh -L 2333:A:5901 A -l Trusty

```
This will forward the A machines' 5901 to local's 2333 port.     
Then use a vncviewer software for accessing local machine's 2333 port:    

```
vncviewer localhost:2333

```

Notice, the virtualbox's is named to vboxgtk in opensuse.    
