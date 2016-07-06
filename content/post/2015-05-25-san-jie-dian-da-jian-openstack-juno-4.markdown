---
categories: ["virtualization"]
comments: true
date: 2015-05-25T20:11:17Z
title: 三节点搭建OpenStack Juno(4)
url: /2015/05/25/san-jie-dian-da-jian-openstack-juno-4/
---

Neutron和nova-network的区别在于，nova-network可以让你在每个instance上部署一种网络类型，适合基本的网络功能。而Neutron则使得你可以在一个instance上部署多种网络类型，并且以插件的方式支持多种虚拟化网络。     

详细的介绍，以后慢慢加，理解吃透了再加上来，这里单单提操作步骤。    

### 准备
数据库准备如下:    

```
root@Controller:~# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 39
Server version: 5.5.43-MariaDB-1ubuntu0.14.04.2 (Ubuntu)

Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE neutron;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'xxxxx';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'xxxxx';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> quit;
Bye

```

### 配置服务

```
root@Controller:~# source admin-openrc.sh
root@Controller:~# keystone user-create --name neutron --pass xxxxx
+----------+----------------------------------+
| Property |              Value               |
+----------+----------------------------------+
|  email   |                                  |
| enabled  |               True               |
|    id    | a6d790e8e86749bba1d27972de8eaae2 |
|   name   |             neutron              |
| username |             neutron              |
+----------+----------------------------------+
root@Controller:~# keystone user-role-add --user neutron --tenant service --role admin
root@Controller:~# keystone service-create --name neutron --type network --description "OpenStack Networking"
+-------------+----------------------------------+
|   Property  |              Value               |
+-------------+----------------------------------+
| description |       OpenStack Networking       |
|   enabled   |               True               |
|      id     | 2f6de710ec414797a4a639c2310c8249 |
|     name    |             neutron              |
|     type    |             network              |
+-------------+----------------------------------+
root@Controller:~# keystone endpoint-create --service-id $(keystone service-list | awk '/ network / {print $2}') --publicurl http://Controller:9696 --adminurl http://Controller:9696 --internalurl http://Controller:9696 --region regionOne
+-------------+----------------------------------+
|   Property  |              Value               |
+-------------+----------------------------------+
|   adminurl  |      http://Controller:9696      |
|      id     | a23132fd0a824aa09f3b1ea72cbb97d2 |
| internalurl |      http://Controller:9696      |
|  publicurl  |      http://Controller:9696      |
|    region   |            regionOne             |
|  service_id | 2f6de710ec414797a4a639c2310c8249 |
+-------------+----------------------------------+

```

### 安装和配置网络组件
安装下列包:    

```
# apt-get install neutron-server neutron-plugin-ml2 python-neutronclient
```

在Controller节点上开始配置:    

```
$ sudo vim /etc/neutron/neutron.conf
[database]
...
connection = mysql://neutron:NEUTRON_DBPASS@Controller/neutron
```

配置rabbitMQ认证方式:     

```

[DEFAULT]
...
rpc_backend = rabbit
rabbit_host = Controller
rabbit_password = RABBIT_PASS
```

配置keystone认证:     

```
[DEFAULT]
...
auth_strategy = keystone

#### 删除已有的keystone authtoken配置方式
[keystone_authtoken]
...
auth_uri = http://Controller:5000/v2.0
identity_uri = http://Controller:35357
admin_tenant_name = service
admin_user = neutron
admin_password = NEUTRON_PASS
```
激活ML2插件(Modular Layer 2), router服务, 和overlapping IP地址:    

```
[DEFAULT]
...
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
```

配置网络，以便在网络拓扑发生变化时告知Compute节点:     

```
[DEFAULT]
...
notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True
nova_url = http://Controller:8774/v2
nova_admin_auth_url = http://Controller:35357/v2.0
nova_region_name = regionOne
nova_admin_username = nova
nova_admin_tenant_id = SERVICE_TENANT_ID
nova_admin_password = NOVA_PASS
```

`SERVICE_TENANT_ID` 通过以下命令来获得:     

```
$ source admin-openrc.sh
$ keystone tenant-get service
+-------------+----------------------------------+
|   Property  |              Value               |
+-------------+----------------------------------+
| description |          Service Tenant          |
|   enabled   |               True               |
|      id     | 08a675be93a04cca8a74159a3eefa288 |
|     name    |             service              |
+-------------+----------------------------------+
```

可以自定义打开verbose选项:    

```
[DEFAULT]
...
verbose = True
```

配置Modular Layer2(ML2)插件:     

在[ml2]部分，激活flat and generic routing encapsulation(GRE) network类型驱动, GRE Tenant网络和OVS机制驱动:    

```
$ sudo vim /etc/neutron/plugins/ml2/ml2_conf.ini
[ml2]
...
type_drivers = flat,gre
tenant_network_types = gre
mechanism_drivers = openvswitch
```

`[ml2_type_gre]` 部分，配置tunnel identifier(id)范围:    

```
[ml2_type_gre]
...
tunnel_id_ranges = 1:1000
```

配置securitygroup部分,     

```
[securitygroup]
...
enable_security_group = True
enable_ipset = True
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
```

配置计算服务使用网络:    

```
$ sudo vim /etc/nova/nova.conf
[DEFAULT]
...
network_api_class = nova.network.neutronv2.api.API
security_group_api = neutron
linuxnet_interface_driver = nova.network.linux_net.LinuxOVSInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver
```

在`[neutron]`部分，配置访问参数:     

```
[neutron]
...
url = http://Controller:9696
auth_strategy = keystone
admin_auth_url = http://Controller:35357/v2.0
admin_tenant_name = service
admin_username = neutron
admin_password = NEUTRON_PASS
```

完成安装：    

```
# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade juno" neutron
# service nova-api restart
# service nova-scheduler restart
# service nova-conductor restart
# service neutron-server restart

```

验证:     

```
root@Controller:~# source admin-openrc.sh
root@Controller:~# neutron ext-list
+-----------------------+-----------------------------------------------+
| alias                 | name                                          |
+-----------------------+-----------------------------------------------+
| security-group        | security-group                                |
| l3_agent_scheduler    | L3 Agent Scheduler                            |
| ext-gw-mode           | Neutron L3 Configurable external gateway mode |
| binding               | Port Binding                                  |
| provider              | Provider Network                              |
| agent                 | agent                                         |
| quotas                | Quota management support                      |
| dhcp_agent_scheduler  | DHCP Agent Scheduler                          |
| l3-ha                 | HA Router extension                           |
| multi-provider        | Multi Provider Network                        |
| external-net          | Neutron external network                      |
| router                | Neutron L3 Router                             |
| allowed-address-pairs | Allowed Address Pairs                         |
| extraroute            | Neutron Extra Route                           |
| extra_dhcp_opt        | Neutron Extra DHCP opts                       |
| dvr                   | Distributed Virtual Router                    |
+-----------------------+-----------------------------------------------+

```

### 配置网络节点
安装以下包:    

```
# apt-get install neutron-plugin-ml2 neutron-plugin-openvswitch-agent neutron-l3-agent neutron-dhcp-agent
```

配置， 首先，删除`/etc/neutron/neutron.conf`里所有的database连接，因为网络节点不需要任何数据库连接。     

```
$ sudo vim /etc/neutron/neutron.conf
[DEFAULT]
...
rpc_backend = rabbit
rabbit_host = controller
rabbit_password = RABBIT_PASS
```

更改keystone认证方式:     

```
[DEFAULT]
...
auth_strategy = keystone
[keystone_authtoken]
...
auth_uri = http://controller:5000/v2.0
identity_uri = http://controller:35357
admin_tenant_name = service
admin_user = neutron
admin_password = NEUTRON_PASS
```

有关`[ml2]`的配置:    

```
[DEFAULT]
...
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
```

接着：    

```
$ sudo vim /etc/neutron/plugins/ml2/ml2_conf
[ml2]
...
type_drivers = flat,gre
tenant_network_types = gre
mechanism_drivers = openvswitch
[ml2_type_flat]
...
flat_networks = external
[ml2_type_gre]
...
tunnel_id_ranges = 1:1000
[securitygroup]
...
enable_security_group = True
enable_ipset = True
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
[ovs]
...
local_ip = INSTANCE_TUNNELS_INTERFACE_IP_ADDRESS
enable_tunneling = True
bridge_mappings = external:br-ex
[agent]
...
tunnel_types = gre
```

配置Layer-3客户端:    

```
$ sudo vim /etc/neutron/l3_agent.ini
[DEFAULT]
...
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
use_namespaces = True
external_network_bridge = br-ex
router_delete_namespaces = True

```

配置DHCP客户端：

```
$ sudo vim /etc/neutron/dhcp_agent.init
[DEFAULT]
...
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
use_namespaces = True
dhcp_delete_namespaces = True

[DEFAULT]
...
dnsmasq_config_file = /etc/neutron/dnsmasq-neutron.conf
```

配置dnsmasq配置文件:   

```
$ sudo vim /etc/neutron/dnsmasq_neutron.conf
dhcp-option-force=26,1454
$ sudo pkill dnsmasq
```

配置metadata客户端:     

```
$ sudo vim /etc/neutron/metadata_agent.ini
[DEFAULT]
...
auth_url = http://controller:5000/v2.0
auth_region = regionOne
admin_tenant_name = service
admin_user = neutron
admin_password = NEUTRON_PASS

[DEFAULT]
...
nova_metadata_ip = controller

[DEFAULT]
...
metadata_proxy_shared_secret = METADATA_SECRET


```

对应的，在控制节点上，配置:    

```
$ sudo vim /etc/nova/nova.conf
[neutron]
...
service_metadata_proxy = True
metadata_proxy_shared_secret = METADATA_SECRET
# service nova-api restart

```

配置Open vSwitch(OVS)服务:    

```
# service openvswitch-switch restart
# ovs-vsctl add-br br-ex
# ovs-vsctl add-port br-ex INTERFACE_NAME
# service neutron-plugin-openvswitch-agent restart
# service neutron-l3-agent restart
# service neutron-dhcp-agent restart
# service neutron-metadata-agent restart
```

验证:     

```
$ source admin-openrc.sh
$ neutron agent-list

```

### 计算节点配置
配置如下;    

```
$ sudo vim /etc/sysctl.conf
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
# sysctl -p
```

安装网络组件：    

```
# apt-get install neutron-plugin-ml2 neutron-plugin-openvswitch-agent
```

配置网络组件：    

```
$ sudo vim /etc/neutron/neutron.conf
[DEFAULT]
...
rpc_backend = rabbit
rabbit_host = controller
rabbit_password = RABBIT_PASS
```

keystone组件:    

```
[DEFAULT]
...
auth_strategy = keystone
[keystone_authtoken]
...
auth_uri = http://controller:5000/v2.0
identity_uri = http://controller:35357
admin_tenant_name = service
admin_user = neutron
admin_password = NEUTRON_PASS
```

`[ml2]`插件:    

```
$ sudo vim /etc/neutron/neutron.conf
[DEFAULT]
...
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
```

继续配置ml2插件:    
```
$ sudo vim /etc/neutron/plugins/ml2/ml2_conf.ini
[ml2]
...
type_drivers = flat,gre
tenant_network_types = gre
mechanism_drivers = openvswitch

[ml2_type_gre]
...
tunnel_id_ranges = 1:1000

[securitygroup]
...
enable_security_group = True
enable_ipset = True
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

[ovs]
...
local_ip = INSTANCE_TUNNELS_INTERFACE_IP_ADDRESS
enable_tunneling = True

[agent]
...
tunnel_types = gre
```

配置OVS服务：    

```
# service openvswitch-switch restart

```

配置计算节点使用网络:    

```
$ sudo vim /etc/nova/nova.conf
[DEFAULT]
...
network_api_class = nova.network.neutronv2.api.API
security_group_api = neutron
linuxnet_interface_driver = nova.network.linux_net.LinuxOVSInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[neutron]
...
url = http://controller:9696
auth_strategy = keystone
admin_auth_url = http://controller:35357/v2.0
admin_tenant_name = service
admin_username = neutron
admin_password = NEUTRON_PASS
```

完成安装:    

```
# service nova-compute restart
# service neutron-plugin-openvswitch-agent restart
```

验证， 在Controller节点上:     

```
$ source admin-openrc.sh
$ neutron agent-list
```


### 创建初始化网络

步骤如下:    
创建外网:    

```
# source admin-openrc.sh
# neutron net-create ext-net --router:external True \
--provider:physical_network external --provider:network_type flat
# neutron subnet-create ext-net --name ext-subnet \
--allocation-pool start=10.77.77.200,end=10.77.77.220 \
--disable-dhcp --gateway 10.77.77.1 10.77.77.0/24
```

租户网络:    

```
$ source demo-openrc.sh
$ neutron net-create demo-net
$ neutron subnet-create demo-net --name demo-subnet \
--gateway 10.10.10.1 10.10.10.0/24
$ neutron router-create demo-router
$ neutron router-interface-add demo-router demo-subnet
$ neutron router-gateway-set demo-router ext-net
```

验证, 因为我们用了10.77.77.200~220作为外网的floating IP, 所以路由器外网的IP应该落在10.77.77.200上，在计算节点上直接ping, 看结果:     

```
dash@PowerfulDash:~$ ping 10.77.77.200
PING 10.77.77.200 (10.77.77.200) 56(84) bytes of data.
64 bytes from 10.77.77.200: icmp_seq=1 ttl=64 time=0.323 ms
64 bytes from 10.77.77.200: icmp_seq=2 ttl=64 time=0.177 ms
64 bytes from 10.77.77.200: icmp_seq=3 ttl=64 time=0.141 ms
```

其实上面的结果是在Host机器上Ping的，因host机器已经有了10.77.77.1地址，理论上，如果能Ping通10.77.77.200，证明Router工作正常。    


### Horizon
在控制节点上安装以下包 ：     

```
# apt-get install openstack-dashboard apache2 libapache2-mod-wsgi memcached python-memcache
```

配置:    

```
# vim /etc/openstack-dashboard/local_settings.py
OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = ['*']
CACHES = {
'default': {
'BACKEND': 'django.core.cache.backends.memcached.
MemcachedCache',
'LOCATION': '127.0.0.1:11211',
}
}
TIME_ZONE = "TIME_ZONE"

```

重启服务:    

```
# service apache2 restart
# service memcached restart
```

最后访问:    
[http://Controller/horizon](http://Controller/horizon) 来看到结果.     
