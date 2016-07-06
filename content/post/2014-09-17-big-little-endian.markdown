---
categories: ["null"]
comments: true
date: 2014-09-17T00:00:00Z
title: Big-Little Endian
url: /2014/09/17/big-little-endian/
---

An elegant way for juding big-endian or little-endian of processor:    

```
eCCM2-root-root> python -c "import sys;sys.exit(0 if sys.byteorder=='big' else 1)"
eCCM2-root-root> echo $?
0
[Trusty@~]$ python -c "import sys;sys.exit(0 if sys.byteorder=='big' else 1)"
[Trusty@~]$ echo $?
1

```
So we could see powerpc is the big-endian, while PC is little-endian.    
