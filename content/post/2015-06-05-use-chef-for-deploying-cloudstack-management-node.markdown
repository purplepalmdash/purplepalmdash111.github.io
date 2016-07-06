---
categories: ["virtualization"]
comments: true
date: 2015-06-05T17:07:22Z
title: Use Chef for deploying CloudStack Management Node
url: /2015/06/05/use-chef-for-deploying-cloudstack-management-node/
---

Following record all of the necessary steps for deploying cloudstack management node on kvm based environment.      

### Get The CookBook
You need following cookbooks:     

```
[kkkk@~/chef-repo/cookbooks]$ ls
apt         cloudstack_wrapper      cookbook_cloudstack_wrapper  line   nfs    rbac       selinux  sudo  yum-mysql-community
cloudstack  cloudstack_wrapper_kvm  learn_chef_apache2           mysql  nginx  README.md  smf      yum
[kkkk@~/chef-repo/cookbooks]$ pwd
/home/kkkk/chef-repo/cookbooks
```

Most of the books could be downloaded from the chef supermarket, while the `cookbook_cloudstack_wrapper` is downloaded from the github, and `cloudstack_wrapper_kvm` is modified from it.     

Note: You have to replace all of the `cloudstack_wrapper::` to `cloudstack_wrapper_kvm::` under the copied folder.     

You have to modify the definition of the 

```
[dash@~/chef-repo/cookbooks]$ cat cloudstack_wrapper_kvm/recipes/management_server.rb

......

# download initial systemvm template
cloudstack_system_template 'kvm' do
  nfs_path    node['cloudstack']['secondary']['path']
  nfs_server  node['cloudstack']['secondary']['host']
  url         node['cloudstack']['systemvm']['kvm']
  db_user     node['cloudstack']['db']['username']
  db_password node['cloudstack']['db']['password']
  db_host     node['cloudstack']['db']['host']
  action :create
end
......

```

### Add Node

Add a new node into the system , then you should do following steps for letting the deployment continue:     

```
$ proxychains4  /opt/chef/embedded/bin/gem install cloudstack_ruby_client
$ sudo apt-get update
$  sudo apt-get install nfs-common
```

Now continue until you could see the final result.     

### Verification
After deployment, visit:    

http://YourIP:8080/client   admin/password

