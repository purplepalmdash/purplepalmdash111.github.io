---
categories: ["Virtualization"]
comments: true
date: 2015-07-28T16:10:49Z
title: WorkTips On LandScape
url: /2015/07/28/worktips-on-landscape/
---

### Installation
Landscape is now in PPA repository, add it via:    

```
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:landscape/15.01
$ sudo apt-get update
$ sudo apt-get install landscape-server-quickstart 
```
During the installation will ask you the configuration of postfix, specify local.    

### Configuration
First time you login into the LandScape Root machine, you have to setup your email
address and your password.       
Then you could visit `https://YourIPAddress` for the configuration page.   

Add the node into the Landscape Root Node.    

In to-be-added node, copy the root node's `/etc/ssl/certs/landscape_server_ca.crt` to
`/etc/landscape`, and modify the following configuration:    

```
# vim /etc/landscape/client.conf
ssl_public_key = /etc/landscape/landscape_server_ca.crt
```
Now register the node into the Root Node.    

```
$ sudo apt-get install landscape-client
$ sudo landscape-config --computer-title "LSNode0" --account-name standalone \
--url https://packer-ubuntu-1404-server/message-system --ping-url \
http://packer-ubuntu-1404-server/ping
```

![/images/2015_07_29_11_27_27_582x380.jpg](/images/2015_07_29_11_27_27_582x380.jpg)   

The following steps is to use or configurating the landscape based cluster
administration.      
