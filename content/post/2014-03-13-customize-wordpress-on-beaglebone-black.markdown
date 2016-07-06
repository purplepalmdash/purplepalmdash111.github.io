---
categories: ["Wordpress"]
comments: true
date: 2014-03-13T00:00:00Z
title: Customize Wordpress on BeagleBone Black
url: /2014/03/13/customize-wordpress-on-beaglebone-black/
---

For Using proxy for Wordpress, we can directly edit the wp-config file:<br />

```
	$ cat wp-config.php
	/** Set following for working behind the proxy **/
	//define('WP_PROXY_HOST', '10.0.0.221');
	//define('WP_PROXY_PORT', '9001');
	#define('WP_PROXY_BYPASS_HOSTS','*.local-intranet');
	##
	define('WP_EMMORY_LIMIT', '64M');

```

A trouble shooting, you have to edit the /etc/resolv.conf to change the default dns server, then your wordpress could reach the network and install new plugins or themes. <br />


