---
categories: ["Virtualization"]
comments: true
date: 2015-12-28T15:25:45Z
title: Re-Scan XenServer NICs
url: /2015/12/28/re-scan-xenserver-nics/
---

Since OpenXenManager could not scan the NICs, we need to scan the newly added NICs
under terminal.    

```
# xe pif-list
# xe host-list
# xe pif-scan host-uuid=a7991728-a86f-4a9c-a163-9b819e444488
```

Via `xe host-list` we could get the host-uuid, then pif-scan it via this host-uuid.    

Now you could see the newly added NICs in openxenmanager:    

![/images/2015_12_28_15_27_30_779x232.jpg](/images/2015_12_28_15_27_30_779x232.jpg)   
