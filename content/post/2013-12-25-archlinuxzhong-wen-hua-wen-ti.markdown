---
categories: ["Linux"]
comments: true
date: 2013-12-25T00:00:00Z
title: ArchLinux中文化问题
url: /2013/12/25/archlinuxzhong-wen-hua-wen-ti/
---

本文将涉及到ArchLinux下中文化问题，主要是关于终端字符和vim中代码格式的细调工作。    
###Vim 配置
####Colorscheme配置：    
下载几个美观的主题：
solarized: [https://github.com/altercation/vim-colors-solarized](https://github.com/altercation/vim-colors-solarized)     
molokai: [https://github.com/tomasr/molokai](https://github.com/tomasr/molokai)   
phd: [http://www.vim.org/scripts/script.php?script_id=3139](http://www.vim.org/scripts/script.php?script_id=3139)    
将其解压开后，拷贝到~/.vim/colors，然后修改~/.vimrc:    

```
	set background=dark
	"set background=bright
	"colorscheme solarized
	colorscheme molokai

```
####Vim字体配置：    
Consolas是一种专门为编程人员设计的字体，这一字体的特性是所有字母、数字与符号均能非常容易辨认，而且所有字符都具有相同的宽度，让编人员看着更舒服。但我们用Consolas在显示程序源码时，不可避免要使用中文注释。而Consolas不支持中文，因此中文默认是使用宋体显示的。当使用10点大小的时候，中文就模糊不清了。如果采用斜体显示注释的话，宋体就更加显得支离破碎。    
在中文显示上，雅黑字体确实不错，但雅黑不是等宽字体，不能用于源码显示。    
使用字体工具将雅黑和Consolas集成在一起后，程序员就可以在Linux环境下的源码中看到优秀的中文显示效果。    
下载地址在 :     
[http://dl.dbank.com/c01bo3a1eo](http://dl.dbank.com/c01bo3a1eo) 

[https://code.google.com/p/uigroupcode/downloads/detail?name=YaHei.Consolas.1.12.zip&can=2&q=](https://code.google.com/p/uigroupcode/downloads/detail?name=YaHei.Consolas.1.12.zip&can=2&q=)    

解压缩后，运行以下命令：

```
	sudo mkdir -p /usr/share/fonts/vista
	sudo cp YaHei.Consolas.1.12.ttf /usr/share/fonts/vista/

```
更改权限:

```
	sudo chmod 644 /usr/share/fonts/vista/*.ttf

```
安装字体:

```
	cd /usr/share/fonts/vista/
	sudo mkfontscale
	sudo mkfontdir
	sudo fc-cache -fv

```

###终端字体配置
更改终端模拟器的字体为Yahei Consolas Hybrid即可
gvim中字体设置： 

```
	set guifont=YaHei\ Consolas\ Hybrid\ 11.5

```
其他的vim细调:

```
	set cursorline  "光标线
	set cursorcolumn  "竖线

```
最后效果如下:    
![/images/effect.jpg](/images/effect.jpg)

### Konsole Setup
Since konsole's QT don't think YaHei Consolas is the fonts, we need manually specify its configuration:     

```
$ cat ~/.kde/share/apps/konsole/Shell.profile
[Appearance]
ColorScheme=Linux
- Font=Monospace,13,-1,2,50,0,0,0,0,0
+ Font=YaHei Consolas Hybrid,11,-1,5,50,0,0,0,0,0
```
Then you should close all of the opened konsoles, re-launch the konsole and you will get the beautiful views of the new fonts.    

In 2015 Aug30, the configuration is changed to:    


```
# vim $HOME/.local/share/konsole/Shell.profile
+ Font=YaHei Consolas Hybrid,11,-1,5,50,0,0,0,0,0
```
