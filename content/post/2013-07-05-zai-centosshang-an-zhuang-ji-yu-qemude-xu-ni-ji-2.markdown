---

comments: true
date: 2013-07-05T00:00:00Z
title: 在CentOS上安装基于qemu的虚拟机(2)
url: /2013/07/05/zai-centosshang-an-zhuang-ji-yu-qemude-xu-ni-ji-2/
---

1\. 启动完毕后客户机无法得到网络地址：

故事的小插曲：由于虚拟机是“偷”跑在某台服务器上，不便在系统路径里留下痕迹，只能使用CPU/内存资源，采取了挂载NFS文件系统的方法，Qemu和vde均自行编译，先于vde编译的Qemu在配置的时候未能激活vde选项，造成客户机中无法找到vde配置的网络。

现象:重新编译qemu,使用下列命令时报错：
```
./configure --prefix=Your_place_here --target-list="i386-softmmu \
x86_64-softmmu  i386-linux-user x86_64-linux-user" --enable-kvm \
--enable-user --enable-vde
```

安装vde2到系统路径后重编译qemu成功。而后
```
$ export LD_LIBRARY_PATH=/usr/lib:$LD_LIBRARY_PATH
```
则可正常启动vde/qemu

2\. 开启Linux内核的forwarding, 这样客户机可以接入Internet.
```
echo 1>/proc/sys/net/ipv4/ip_forward 
```


3\. 增加更多的核:

在启动参数中添加:
```
-cpu host -smp cores=2,threads=1 
```

To be continued.
