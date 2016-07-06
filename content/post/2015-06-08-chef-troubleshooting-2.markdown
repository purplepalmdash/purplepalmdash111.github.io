---
categories: ["Linux"]
comments: true
date: 2015-06-08T19:43:13Z
title: Chef TroubleShooting 2
url: /2015/06/08/chef-troubleshooting-2/
---

The ssl checking for adding new nodes is still a horrible procedure, following shows the correct steps for adding new node:    

On Chef Workstation, add node via its ip address rather than via its hostname:    

```
$ knife bootstrap 172.16.0.12 -x username_on_12 -P password_on_12 --sudo
```

The example knife.rb file should be written like following:    

```
current_dir = File.dirname(__FILE__)
log_level                :info
log_location             STDOUT
node_name                "nodename"
client_key               "#{current_dir}/node.pem"
validation_client_name   "nodename"
validation_key           "#{current_dir}/node_org.pem"
chef_server_url          "https://tmpChefServer/organizations/nodename"
syntax_check_cache_path  "#{ENV['HOME']}/.chef/syntaxcache"
cookbook_path            ["#{current_dir}/../cookbooks"]
```

On Chef Client, first fetching the ssl, then manually passed the verification.     

```
$ knife ssl fetch --config /etc/chef/client.rb
$ chef-client -l debug -S https://ChefServer/organizations/xxxxx -K /xxx/xxx/xxxxx.pem
```

On Chef Server, bootstrap again via the same command, now you could work.    
