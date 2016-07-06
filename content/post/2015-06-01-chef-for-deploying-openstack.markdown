---
categories: ["linux"]
comments: true
date: 2015-06-01T11:19:34Z
title: Chef For Deploying OpenStack
url: /2015/06/01/chef-for-deploying-openstack/
---

Following article records all of the steps for using chef for deploying OpenStack.    

Refers to:    
[http://ehaselwanter.com/en/blog/2014/10/15/deploying-openstack-with-stackforge-chef-zero-style/](http://ehaselwanter.com/en/blog/2014/10/15/deploying-openstack-with-stackforge-chef-zero-style/)    


### Change vbox files
Edit the Vagrantfile for bring up the vbox, then startup the machine, modify its content , save it.    

```
$ vim Vagrantfile
    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    Vagrant::Config.run do |config|
    config.vm.box = "Trusy64"
    config.vm.box_url = "http://xxx.xxx.xxx.xxx/opscode_ubuntu-14.04_chef-provisionerless.box"
    config.vm.customize ["modifyvm", :id, "--memory", 1024]
    end
```
Login to the running machine and modify its default repository from official to local repository.    

```
$ vagrant up
$ vagrant ssh
(YourVagrantMachine) $ sudo vim /etc/apt/sources.list
(YourVagrantMachine) $ sudo vim /etc/apt/apt.conf
(YourVagrantMachine) $ sudo apt-get update && sudo apt-get -y upgrade
```

Now save your modification to the vbox file:    

```
$ vagrant package --base vagrant_default_1433130468275_38998
$ ls
package.box Vagrantfile
```

### Setup Chef Code
First install the vagrant plugins via:    

```
$ vagrant plugin install vagrant-berkshelf
$ vagrant plugin install vagrant-chef-zero
$ vagrant plugin install vagrant-omnibus
$ vagrant plugin list
```

Get the repository from github, modify the file `vagrant_linux.rb`:    

```
[xxxx@~/Code/Chef/MasterVersion]$ git clone https://github.com/stackforge/openstack-chef-repo.git
$ cd openstack-chef-repo
$ vim vagrant_linux.rb
  #url 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_centos-7.1_chef-provisionerless.box'
  url 'http://xxx.xxx.xxx.xxx/opscode_centos-7.1_chef-provisionerless.box'

  #url 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_ubuntu-14.04_chef-provisionerless.box'
  url 'http://xxx.xxx.xxx.xxx/package.box'

  'vm.box' => 'ubuntu14'
```
Download all of the cookbooks, and modify the rubygems.org to Chinese mirror, Thanks for the fucking GreatFW!:    

```
$ chef exec rake berks_vendor
$ cp -r cookbooks cookbooks.back
$ cd cookbooks
$ find . -type f -exec sed -i -e 's/https:\/\/rubygems.org/http:\/\/mirrors.aliyun.com\/rubygems/g' {} \; 
```
Edit the ruby definition file for avoiding ` Chef encountered an error attempting to load the node data for "controller"`:    

```
$ vim ./aio-neutron.rb
machine 'controller' do
  add_machine_options vagrant_config: controller_config
+  chef_server( :chef_server_url => 'http://localhost:8889')
  role 'allinone-compute'
  role 'os-image-upload'

```


One Cookbook needs to modify, because it will automatically use source from `rubygems.org`, Thanks again for the fucking GreatFW!:    

```
$ cd cookbooks
$ vim ./mysql2_chef_gem/libraries/provider_mysql2_chef_gem_mysql.rb
             options("--clear-sources --source http://mirrors.aliyun.com/rubygems/gems/mysql2-0.3.18.gem") 

```


Now begin to provision via:    

```
$ chef exec rake aio_neutron 2>&1 | tee aio_neutron.txt
```

After installation and configuration, you could visit following URL for your OpenStack:     

https://127.0.0.1:9443
