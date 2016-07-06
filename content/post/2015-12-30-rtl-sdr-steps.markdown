---
categories: ["radio"]
comments: true
date: 2015-12-30T16:52:48Z
title: rtl-sdr steps
url: /2015/12/30/rtl-sdr-steps/
---

Installing and configurating steps:    

```
$ git clone https://github.com/balint256/gr-baz
$ sudo apt-get install libboost-all-dev gr-osmosdr gnuradio-dev liblog4cpp5-dev 
$ cd gr-baz
$ mkdir build
$ cd build
$ cmake ..
$ make && sudo make install
```

Install rtl-sdr:    

```
$ git clone git://git.osmocom.org/rtl-sdr.git 
$ cd rtl-sdr 
$ mkdir build
$ cd build/
$ cmake ../ -DINSTALL_UDEV_RULES=ON
$ sudo make install
$ sudo ldconfig 
```
Now you could use `sudo rtl_eeprom` for probing the rtl equipments.  


```
$ sudo modprobe -r dvb_usb_rtl28xxu
$ sudo apt-get install -y gqrx-sdr
```

Using `gqrx` could scan the frequency and get the radio stations.   
