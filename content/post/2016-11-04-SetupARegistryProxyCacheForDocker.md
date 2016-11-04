+++
categories = ["Linux"]
date = "2016-11-04T17:14:21+08:00"
description = "Setup a docker registry proxy cached server"
keywords = ["Docker"]
title = "SetupARegistryProxyCacheForDocker"

+++
### 准备
首先需要准备以下Docker image:    

```
$ sudo docker pull registry:latest
$ sudo docker pull registry:2
```
在`/etc/hosts`里加一条记录，接下来我们将用这条记录生成签名(注意修改IP地址和域名）:    

```
# echo '192.168.0.121    xxx.xxx.xxx.com'>>/etc/hosts
```

准备所需要的自签名文件(用于tls连接)    

```
mkdir -p certs && openssl req   -newkey rsa:4096 -nodes -sha256 -keyout
certs/domain.key   -x509 -days 365 -out certs/domain.crt
Generating a 4096 bit RSA private key
....................................................................................................++
......++
writing new private key to 'certs/domain.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Guangdong
Locality Name (eg, city) []:Guangzhou
Organization Name (eg, company) [Internet Widgits Pty Ltd]:DashCom
Organizational Unit Name (eg, section) []:IT
Common Name (e.g. server FQDN or YOUR name) []:xxx.xxx.xxx.com
Email Address []:xxxxxx@gmail.com
```
### 启动容器
首先拷贝出默认的registry容器的配置文件:    

```
$ sudo mkdir -p /data
$ sudo chmod 777 -R /data
$ sudo docker run -it --rm --entrypoint cat registry:2 \
/etc/docker/registry/config.yml > /data/config.yml
```
修改默认文件，添加`proxy`字段等，我的配置如下:    

```
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  tls:
    certificate: /var/lib/registry/domain.crt
    key: /var/lib/registry/domain.key
  headers:
    X-Content-Type-Options: [nosniff]
proxy:
  remoteurl: https://registry-1.docker.io
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```
将上一步生成的`domain.crt`和`domain.key`文件也拷贝到/data文件夹下.    

现在运行容器:    

```
$ sudo docker run -d --restart=always -p 5000:5000 --name v2-mirror  -v
/data:/var/lib/registry registry:2 /var/lib/registry/config.yml
```
运行完这条命令后，可以用`sudo netstat -anp| grep 5000`来检查registry服务是否被启动。    

### 使用Registry服务
在客户机上，需要做以下设置, 这里我写成脚本的方式，具体的调整需要根据实际情况:    

```
echo '192.168.0.121    xxx.xxx.xxx.com'>>/etc/hosts
apt-get install -y wget
mkdir -p /etc/docker/certs.d/xxx.xxx.xx.com:5000/
wget -O /etc/docker/certs.d/xxx.xxx.xxx.com:5000/ca.crt http://192.168.0.121/domain.crt
sed -i 's#fd://#fd:// --registry-mirror https://xxx.xxx.xxx.com:5000#' /lib/systemd/system/docker.service
systemctl daemon-reload
systemctl restart docker
```
这里的`http://192.168.0.121/domain.crt`是我把domain.crt文件放在一个web服务器上了，可以
直接拷贝现成的，改名即可。    

现在测试：    

```
# curl -I https://xxx.xxx.xxx.com:5000/v2 --cacert \ 
/etc/docker/certs.d/xxx.xxx.xxx.com\:5000/ca.crt 

HTTP/1.1 301 Moved Permanently
Docker-Distribution-Api-Version: registry/2.0
Location: /v2/
Date: Fri, 04 Nov 2016 09:42:41 GMT
Content-Type: text/plain; charset=utf-8
```
检查镜像:    

```
# curl  https://xxx.xxx.xxx.com:5000/v2/_catalog --cacert \
/etc/docker/certs.d/xxx.xxx.xxx.com\:5000/ca.crt 
{"":[""]}
```
docker pull一个镜像:    

```
# docker pull busybox:latest
```
现在检查:    

```
# curl  https://xxx.xxx.xxx.com:5000/v2/_catalog --cacert \
/etc/docker/certs.d/xxx.xxx.xxx.com\:5000/ca.crt 
{"repositories":["library/busybox","library/ubuntu"]}
```
可以看到，library/busybox这个镜像已经被缓存在了本地registry仓库中。    
