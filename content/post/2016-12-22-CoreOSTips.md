+++
categories = ["Virtualization"]
date = "2016-12-22T18:02:47+08:00"
description = "CoreOS tips"
keywords = ["Virtualization"]
title = "CoreOSTips"

+++
### Add additional ssh keys
Adding new keys into the deployed system:    

```
# echo 'ssh-rsa AAAAB3Nza.......  key@host' | update-ssh-keys -a core
```
### Write files
Take `/etc/environment` file for example:    

```
core@coreos1 ~ $ cat /usr/share/oem/cloud-config.yml 
#cloud-config
write_files:
    - path: /etc/environment
      permissions: 0644
      content: |
        COREOS_PUBLIC_IPV4=172.17.8.201
        COREOS_PRIVATE_IPV4=172.17.8.201
```

