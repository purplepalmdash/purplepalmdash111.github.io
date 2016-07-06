---
categories: ["Virtualization"]
comments: true
date: 2015-08-05T11:28:21Z
title: Enable DHCP/DNS Server For SpaceWalker Server
url: /2015/08/05/enable-dhcp-slash-dns-server-for-spacewalker-server/
---

### DHCP Server
Install the dhcp server via:   

```
# yum install -y dhcp
```
Then edit the configuration of `/etc/dhcp/dhcpd.conf`, like following:    

```
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp*/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
# specify name server's hostname or IP address
option domain-name-servers 10.9.10.13;
# default lease time
default-lease-time 600;
# max lease time
max-lease-time 7200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnet mask
subnet 10.9.10.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.9.10.200 10.9.10.254;
    # specify broadcast address
    option broadcast-address 10.9.10.255;
    # specify default gateway
    option routers 10.9.10.1;
    # Specify default dns server
    option domain-name-servers 10.9.10.13;
}
```
Start the dhcpd server via:     

```
# service dhcpd start
```

### DNS Server(Bind9)    
Just serve the private network. Install the bind9 via:      

```
# yum install -y bind bind-utils
```
Our server's name is `spacewalker`, need to map to `10.9.10.13`, following is the
configuration steps:     
Edit the file of `/etc/named.conf`.    

Change the listen address of port 53:    

```
options {
        listen-on port 53 { 127.0.0.1; 10.9.10.13; };
#        listen-on-v6 port 53 { ::1; };
...
```
Also add the allow-query items, to let the local network nodes for querying its dns:    

```
 allow-query     { localhost; 10.9.10.0/24;};
```
Add a new zone named `spacewalker`:    

```
zone "spacewalker" {
        type master;
        file "/etc/named/zones/db.spacewalker";
};
```
Now add the zone definition file:    

```
# vim /etc/named/zones/db.spacewalker
$TTL 604800
@       IN      SOA     spacewalker. root.spacewalker. (
                                3               ;Serial
                                604800          ;Refresh
                                86400           ;Retry
                                2419200         ;Expire
                                604800 )        ;Negative Cache TTL
;
; name servers - NS records
      IN      NS      spacewalker.

; name servers  - A records
spacewalker.         IN      A       10.9.10.13
```
Check the configuration file format:     

```
# sudo named-checkconf
# sudo named-checkzone spacewalker /etc/named/zones/db.spacewalker 
zone spacewalker/IN: loaded serial 3
OK
```
Start bind9 service via:     

```
# systemctl start named
```

If in CentOS6, then the steps may like following:    

```
[root@spacewalk named]# service named start
Generating /etc/rndc.key:                                  [  OK  ]
Starting named:                                            [  OK  ]
[root@spacewalk named]# chkconfig --level 345 named on
[root@spacewalk named]# chkconfig --list named
named           0:off   1:off   2:off   3:on    4:on    5:on    6:off
```

You could verify the correctness by startup a live-cd and view its ping result of `ping
spacewalker`.    

