---
categories: ["linux"]
comments: true
date: 2014-05-21T00:00:00Z
title: Moving From Working PC To Own USB-Disk Based 2
url: /2014/05/21/moving-from-working-pc-to-own-usb-disk-based-2/
---

### Install Software
####  Internet
Chromium, we need this browser absolutesly:   

```
$ sudo pacman -S chromium
Select 1/8

```
Oh, we forget install X, so first we will install X: 
#### X Window
Install xorg first:   

```
$ sudo pacman -S xorg xorg-xinit

```
Awesome Window Manager;    

```
$ sudo pacman -S awesome

```
Edit the .xinitrc file, add following lines:    

```
exec awesome

```
Necessary video driver:    

```
$ sudo pacman -S xf86-video-intel xf86-video-ati

```
#### Continue Internet
Firefox, another browser, pidgin, for chatting, thunderbird for email, wget for downloading:    

```
sudo pacman -S firefox pidgin thunderbird wget

```
#### Office
Libreoffice,need download 253MB, so take a break:    

```
$ sudo pacman -S libreoffice
# choose Enter-> 25-> Enter

```

#### Terminals
Install xterm first, initial term:

```
$ sudo pacman -S xterm

```
xfce4-terminal will be installed in next chapter`Vncserver`.     
gnome-terminal is a good option:   

```
$ sudo pacman -S gnome-terminal

```



#### Vncserver
Install tigervnc 

```
$ sudo pacman -S tigervnc

```
Use xfce4 for remote vnc(For many users may not familiar with awesome desktop):  

```
$ sudo pacman -S xfce4 
# choose (default=all)

```
Configure vncserver:

```
$ cat ~/.vnc/xstartup 
#!/bin/sh

export XKL_XMODMAP_DISABLE=1
exec startxfce4

```
With VNC, you can start up the X and let the chromium sync up its bookmarks.     
#### Documents
Install evince for viewing pdf and other documents:

```
$ sudo pacman -S evince

```
Install gimp for processing pictures:    

```
$ sudo pacman -S gimp

```

#### Video/Fun
Install smplayer:    

```
sudo pacman -S smplayer

```
Without ALSA, you can do nothing, so install it. 

```
sudo pacman -S alsa-utils

```

#### Development
Install gvim and eclipse:    

```
$ sudo pacman -S gvim eclipse

```
Git and Subversion:    

```
$ sudo pacman -S git subversion

```
Wireshark and Tcpdump:    

```
$ sudo pacman -S wireshark-gtk tcpdump

```
ddd and gdb for debugging:    

```
$ sudo pacman -S ddd gdb

```
meld for comparing files:    

```
$ sudo pacman -S meld

```



####  Virtualization
Install qemu and virtualbox:     

```
sudo pacman -S virtualbox qemu

```


#### Chinese Localization
Install fonts for chinese:    

```
$ sudo pacman -S wqy-bitmapfont wqy-microhei wqy-microhei-lite wqy-zenhei

```
Install fcitx input method:    

```
$ sudo pacman -S fcitx fcitx-libpinyin fcitx-configtool fcitx-qt4 fcitx-qt5

```
Configuration will be wrote later.    

#### System Tools
rox for file browser:    

```
$ sudo pacman -S rox

```
gpicview for picture viewer:    

```
$ sudo pacman -S gpicview

```
Conky for viewing the tasks:    

```
$ sudo pacman -S conky

```
.conky file will be downloaded from the github(Later I will upload). 
.xinitrc file will be uploaded to github. 



Now step2 is OK, we will use usd-disk for rebooting.    
