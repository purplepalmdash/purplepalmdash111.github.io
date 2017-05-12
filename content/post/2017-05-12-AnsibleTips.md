+++
categories = ["Linux"]
keywords = ["Linux"]
description = "Ansible tips for using"
title = "Ansible Tips on register and so on"
date = "2017-05-12T12:25:17+08:00"

+++
1. 处理用户输入


```
vars_prompt, 设置KVMPassword的值为多少

- name: apply CloudStack configuration to all nodes
  hosts: cloudstackmanagement
  sudo: yes

  #########################################################################################
  # Vars and vars_prompt
  #
  vars_prompt:

    - name: "KVMPassword"
      prompt: "KVM host root password"
      private: yes
```

2. 可以使用when来界定用户输入值的有效

```
    - name: Validate input - KVMServer host password
      fail: msg="Missing or incorrect KVM Host password."
      when: KVMPassword is not defined or ( KVMPassword is defined and KVMPassword  == "" )
```

when 针对不同的返回值采取不同的行动（举某个安装命令yum/apt差别为例）


```
- name: Install the required  packages in Redhat derivatives
  yum: name={{ item }} state=installed
  with_items: network_pkgs
  when: ansible_os_family == 'RedHat'

- name: Install the required packages in Debian derivatives
  apt: name={{ item }} state=installed update_cache=yes
  with_items: network_pkgs
  environment: env
  when: ansible_os_family == 'Debian'
```

3. 某节点输出的值可以被写入到register中，譬如以下命令根据预定义参数，创建zone，管道输出后结果到ZoneID变量中。

```
    - name: Configure zone and add resources
      shell: cloudmonkey create zone name={{ CMConfig.ZoneName }} dns1={{ CMConfig.PublicDNS1 }} dns2={{ CMConfig.PublicDNS2 }} internaldns1={{ CMConfig.InternalDNS1 }} internaldns2={{ CMConfig.InternalDNS2 }} guestcidraddress={{ CMConfig.GuestCIDR }} networktype={{ CMConfig.NetworkType }} localstorageenabled=false | grep ^id | awk '{print $3}'
      register: ZoneID
```

4. 对ZoneID变量的使用:  {{ ZoneID.stdout }}

```
    - name: Create physical network 1
      shell:  cloudmonkey create physicalnetwork name={{ CMConfig.Phys1Name }} zoneid={{ ZoneID.stdout }} isolationmethods={{ CMConfig.Phys1Isolation }} vlan={{ CMConfig.Phys1VLANs }} | grep ^id | awk '{print $3}'
      register: Phys1ID
```
