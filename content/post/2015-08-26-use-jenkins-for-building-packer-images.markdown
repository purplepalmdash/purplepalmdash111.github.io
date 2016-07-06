---
categories: ["null"]
comments: true
date: 2015-08-26T16:36:20Z
title: Use Jenkins For Building Packer Images
url: /2015/08/26/use-jenkins-for-building-packer-images/
---

### Installation Jenkins
Refers to
[https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu)     

```
$ wget -q -O - https://jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
$ sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
$ sudo apt-get update
$ sudo apt-get install -y jenkins
```

### Install Packer Plugins
Manually(But failed), so finally I use the web-backed for installing.   

```
$ wget https://ci.jenkins-ci.org/jnlpJars/jenkins-cli.jar
$ java -jar jenkins-cli.jar -s http://localhost:8080 help
$ java -jar jenkins-cli.jar -s http://localhost:8080 list-plugins
$ java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin \ 
http://ftp.yz.yamagata-u.ac.jp/pub/misc/jenkins/plugins/packer/1.2/packer.hpi
```

![/images/2015_08_26_16_56_22_672x344.jpg](/images/2015_08_26_16_56_22_672x344.jpg)     

After installation, restart service via `service restart jenkins`.    

How to use it? later will investigate.    
