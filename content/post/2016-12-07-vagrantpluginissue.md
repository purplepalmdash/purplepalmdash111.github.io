+++
categories = ["Linux"]
date = "2016-12-07T22:00:18+08:00"
description = "vagrant issue after update"
keywords = ["Linux"]
title = "vagrantpluginissue"

+++
After updating vagrant, my vagrant could use, the error msg are listed as
following:    

```
$ vagrant box list
Vagrant failed to initialize at a very early stage:

The plugins failed to initialize correctly. This may be due to manual
modifications made within the Vagrant home directory. Vagrant can
attempt to automatically correct this issue by running:

  vagrant plugin repair

If Vagrant was recently updated, this error may be due to incompatible
versions of dependencies. To fix this problem please remove and re-install
all plugins. Vagrant can attempt to do this automatically by running:

  vagrant plugin expunge --reinstall

```
I tried `vagrant plugin repair` and `vagrant plugin expunge --reinstall` but
both failed.     

So , manually reinstall the vagrant-libvirt plugin:    

```
$  CONFIGURE_ARGS='with-ldflags=-L/opt/vagrant/embedded/lib \
with-libvirt-include=/usr/include/libvirt with-libvirt-lib=/usr/lib' \
  GEM_HOME=~/.vagrant.d/gems GEM_PATH=$GEM_HOME:/opt/vagrant/embedded/gems  \
PATH=/opt/vagrant/embedded/bin:$PATH \
  vagrant plugin install vagrant-libvirt

Installing the 'vagrant-libvirt' plugin. This can take a few minutes...
/opt/vagrant/embedded/lib/ruby/2.2.0/rubygems/dependency.rb:315:in `to_specs':
Could not find 'fog-core' (>= 0) among 44 total gem(s) (Gem::LoadError)
Checked in 'GEM_PATH=/opt/vagrant/embedded/gems', execute `gem env` for more
information
```
Seems we need to reinstall fog-core to vagrant's `GEM_PATH`, use following
commands:    

```
$ sudo pacman -S ruby
$ gem source -r https://rubygems.org
$ gem source -a http://rubygems.org
$ GEM_HOME=~/.vagrant.d/gems GEM_PATH=$GEM_HOME:/opt/vagrant/embedded/gems \
PATH=/opt/vagrant/embedded/bin:$PATH
$ sudo chmod 777 -R /opt/vagrant/embedded/gems
$ vim ~/.gemrc
gem: "--user-install"
+ install: --no-user-install
$ proxychains4 gem install --install-dir /opt/vagrant/embedded/gems/ fog-core
$  CONFIGURE_ARGS='with-ldflags=-L/opt/vagrant/embedded/lib \
with-libvirt-include=/usr/include/libvirt with-libvirt-lib=/usr/lib' \
  GEM_HOME=~/.vagrant.d/gems GEM_PATH=$GEM_HOME:/opt/vagrant/embedded/gems  \
PATH=/opt/vagrant/embedded/bin:$PATH \
  vagrant plugin install vagrant-libvirt
$ vim ~/.gemrc
gem: "--user-install"
- install: --no-user-install
```
Now vagrant will work ok, verify it via:    

```
$ vagrant box list
CentOS72ForKubernetes                 (virtualbox, 0)
CentOS72KVM                           (libvirt, 0)
CentOS72Kubernetes                    (virtualbox, 0)
```
