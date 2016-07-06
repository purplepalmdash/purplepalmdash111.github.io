---
categories: ["github"]
comments: true
date: 2014-07-30T00:00:00Z
title: 在github上部署你的octopress博客
url: /2014/07/30/zai-githubshang-bu-shu-ni-de-octopressbo-ke/
---

环境: ArchLinux    
### 准备
ArchLinux上安装git 和 ruby:    

```
$ sudo pacman -S git ruby 

```
安装rvm, 因为我用代理的缘故，所以使用了`-k --insecure` 选项，如果你的网络未收到任何阻挠，推荐你使用`curl -L https://get.rvm.io | bash -s stable --ruby`:      

```
$ echo insecure >> ~/.curlrc
$ curl -k --insecure  -L https://get.rvm.io | bash -s stable --ruby

```
重新登录终端：    

```
[root@archi386 ~]# which rvm
/usr/local/rvm/bin/rvm
[root@archi386 ~]# rvm install 1.9.3
[root@archi386 ~]# rvm use 1.9.3
Using /usr/local/rvm/gems/ruby-1.9.3-p547
[root@archi386 ~]# rvm rubygems latest
Rubygems 2.2.2 already available in installed ruby, skipping installation, use --force to reinstall.

```
在上面的步骤后，你可以运行下列命令，确认自己的ruby版本是1.9.3    

```
[root@archi386 ~]# ruby --version
ruby 1.9.3p547 (2014-05-14 revision 45962) [i686-linux]

```

在你定义的目录里, 安装Octopress:     

```
$ git clone git://github.com/imathis/octopress.git octopress
$ cd octopress
$ gem install bundler
$ bundle install

```
接下来，安装默认的主题:    

```
rake install

```
补充：在ArchLinux上，需要更改Gemfile: 添加下面两行：    

```
  gem 'therubyracer'
  gem 'execjs'

```
而后执行`bundle install`后方可运行`rake generate`.     

### Github
申请新的github帐号：    
到[https://github.com/join](https://github.com/join)申请你的github帐号， 提供一个例子如下：     
![/images/githubjoin.jpg](/images/githubjoin.jpg)

创建一个新的代码仓库:    
![/images/createnew.jpg](/images/createnew.jpg)    
取名：    
![/images/reponame.jpg](/images/reponame.jpg)    

如果要使用github的托管网页服务，需要让你的邮箱通过验证。       

### 部署博客
运行下列命令后，照着格式填写诸如'git@github.com:maofphn/maofphn.github.io.git`的地址。    

```
$ rake setup_github_pages

```
生成和部署博客:    

```
$ rake generate
$ rake deploy

```
而后访问[http://maofphn.github.io](http://maofphn.github.io)就可以看到生成的网站了。    
要写新博文，用下列命令:    

```
$ rake new_post["文章标题"]

```
生成的是markdown文件，所以你需要熟悉markdown语法，这里有个作弊手册:    
[https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)


### Trouble Shooting
In ubuntu, you need to intall `ruby-dev` first, then you could use `bundle install` for building the local repository.      
Or you will met:    

```
Error: failed to build gem native extension

```
If you met following error,      

```
YAML Exception reading index.markdown: invalid byte sequence in US-ASCII

```
Then you should do following:    

```
# For using bundle
export LANG=en_US.UTF-8
export LANGUAGE=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```
