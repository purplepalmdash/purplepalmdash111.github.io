---

comments: true
date: 2013-07-12T00:00:00Z
title: Qemu快速保存和恢复状态
url: /2013/07/12/qemukuai-su-bao-cun-he-hui-fu-zhuang-tai/
---

1\. 启动镜像：

```
	$ qemu-system-i386 -hda hd.qcow2
```

2\. 保存当前运行状态：

同时按下ctrl+alt+2切换到Qemu内建命令行，输入:

```
	(qemu) savevm booted
```

如果需要即时回复到保存时状态

```
	(qemu) loadvm booted
```

关闭Qemu运行窗口

3\. 快速恢复到保存状态:

```
	$ qemu-system-i386 -hda hd.qcow2 -loadvm booted
```


