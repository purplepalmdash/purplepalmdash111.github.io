+++
categories = ["Linux"]
date = "2016-11-03T17:23:23+08:00"
description = "Speed up docker deployment"
keywords = ["Linux"]
title = "DockerSpeedUp"

+++
### Docker Registry
因为众所周知的原因，在国内下载Docker镜像会很慢，所以我们更改docker的配置，让它使用
daocloud的加速服务:    

ArchLinux下，编辑以下文件，或者你可以通过`sudo systemctl edit docker.service`来配置以下
文件:    

```
# vim /etc/systemd/system/docker.service.d/override.conf 
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --registry-mirror=http://1a653205.m.daocloud.io -H fd://
```
现在重新家在服务并重新启动`docker.service`：    

```
# systemctl daemon-reload
# systemctl restart docker.service
```
从此以后，每一次docker pull都会使用到daocloud提供的加速服务。    

### 本地私有registry
首先pull下来如下镜像(这两个其实是一个镜像):    

```
# docker pull registry:latest
# docker pull registry:2
```
运行registry实例(注意更改volumn映射):    

```
# docker run -d -p 5000:5000 --restart=always --name registry -v
/root/DockerRegistry:/var/lib/registry registry:2
```
检查实例运行情况:    

```
➜  ~ sudo docker ps
CONTAINER ID        IMAGE                            COMMAND                  CREATED
STATUS              PORTS                    NAMES
b16b27933709        registry:2                       "/entrypoint.sh /etc/"   7 hours
ago         Up 2 hours          0.0.0.0:5000->5000/tcp   registry
```
为了让这个服务每次都启动，我们来编写一个systemd服务:    

```
# vim /usr/lib/systemd/system/DockerRegistry.service 
[Unit]
Description=DockerRegistry container
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker start -a registry
ExecStop=/usr/bin/docker stop -t 2 registry

[Install]
WantedBy=multi-user.target
```
现在使能该服务:    

```
# systemctl enable DockerRegistry.service
Created symlink /etc/systemd/system/multi-user.target.wants/DockerRegistry.service →
/usr/lib/systemd/system/DockerRegistry.service.
```
### 本地私有Registry使用方法
将获取到的image tag到私有registry(tag):   

```
# docker tag ubuntu:16.04 localhost:5000/ubuntu:16.04
```
将获取到的image push到私有registry(push):    

```
# docker push localhost:5000/ubuntu:16.04
```

