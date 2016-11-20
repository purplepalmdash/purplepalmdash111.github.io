+++
categories = ["Linux"]
date = "2016-11-19T08:57:36+08:00"
description = "Some tips in Ansible"
keywords = ["Linux"]
title = "SomeTipsOnAnsible"

+++
In this series I will collect some of the tips in using ansible for playing
automation deployment.    
### co-operation with vagrant ssh
Method: In inventory file, add vagrant's ssh key.   
First you should get the ssh indentity file via:    

```
$ vagrant ssh-config | grep IdentityFile
# result should be your private key and not
#   .vagrant/machines/default/virtualbox/private_key
```
Add these file definition into your inventory file:    

```
[master]
192.168.33.17 ansible_ssh_port=22 ansible_ssh_user=vagrant ansible_ssh_private_key_file=/var1/Nov14/test/.vagrant/machines/master/virtualbox/private_key

[node1]
192.168.33.18 ansible_ssh_port=22 ansible_ssh_user=vagrant ansible_ssh_private_key_file=/var1/Nov14/test/.vagrant/machines/node1/virtualbox/private_key
```
now you will use vagrant and its indentity file for accessing the nodes.    

### Ignore ssh authenticity
Make a file named `ansible.cfg` under your deployment folder:    

```
$ vim ansible.cfg
[defaults]
host_key_checking = False
```
Or use the environment variables for deploying:     

```
$ ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory testall.yml
```
### become method
The usage of `sudo` is not be encouraged for using, instead we use:    

```
- hosts: all
  become: true
  become_user: root
  gather_facts: no
  remote_user: vagrant
```
### touch method
To make sure the file is touched, before we use `touch somefile`, now we use:    

```
  tasks:
    - name: touch something in the /tmp
      file: path=/tmp/abc.txt state=touch
```
