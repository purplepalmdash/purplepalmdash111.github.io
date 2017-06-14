+++
description = "Working tips on gitlab ci"
title = "WorkingTipsOnGitLabCI"
categories = ["Linux"]
keywords = ["Linux"]
date = "2017-06-14T08:30:37+08:00"

+++
### external_url
You should configure the `external_url` to your specified IP address, or you
won't successfully pulling codes into your working nodes:    

```
$ vim /etc/gitlab/gitlab.rb
external_url 'http://192.168.33.2'
$ gitlab-ctl reconfigure
$ gitlab-ctl restart
```
Now you could successfully pulling codes into your working nodes.    
