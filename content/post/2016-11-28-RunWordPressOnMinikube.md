+++
categories = ["Virtualization"]
date = "2016-11-28T11:53:25+08:00"
description = "Run Wordpress On Minikube"
keywords = ["Virtualization"]
title = "RunWordPressOnMinikube"

+++
### Installation
On Ubuntu16.04, first download the deb package from    

[https://github.com/kubernetes/minikube/releases](https://github.com/kubernetes/minikube/releases)    

Install virtualbox:    

```
$ sudo apt-get install -y virtualbox
$ sudo dpkg -i minikube_0.12-2.deb
$ which minikube-linux-amd64 
/usr/bin/minikube-linux-amd64
```
### Start Cluster
First install kubectl:    

```
$ curl -Lo kubectl \
https://storage.googleapis.com/kubernetes-release/release/v1.3.0/bin/linux/amd64/kubectl \
&& chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```
Start kubernetes cluster via:    

```
$ minikube-linux-amd64 start
Starting local Kubernetes cluster...
Downloading Minikube ISO
 36.00 MB / 36.00 MB [==============================================] 100.00%
0s
Kubectl is now configured to use the cluster.
```
Examine the result:    

```
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                          READY     STATUS RESTARTS   AGE
kube-system   kube-addon-manager-minikube   0/1       ContainerCreating   0	1m
```
Examine the status:    

```
$ minikube-linux-amd64 status
minikubeVM: Running
localkube: Running
```

### Trouble-Shooting In Dashboard
When startup the dashboard, the minikube will complains could not find the endpoint:    

```
$ minikube-linux-amd64 dashboard
Could not find finalized endpoint being pointed to by kubernetes-dashboard: Temporary Error: endpoints "kubernetes-dashboard" not found
Temporary Error: endpoints "kubernetes-dashboard" not found
Temporary Error: endpoints "kubernetes-dashboard" not found
Temporary Error: endpoints "kubernetes-dashboard" not found
```
Solved:    
Get all of the pods in all namespaces:    

```
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                          READY     STATUS              RESTARTS   AGE
default       nginx-3449338310-vna7q        0/1       ContainerCreating   0          2h
kube-system   kube-addon-manager-minikube   0/1       ContainerCreating   0          3h
```
Get the description of the pod `kube-addon-manager-minikube`:    

```
$ kubectl describe --namespace=kube-system po kube-addon-manager-minikube
Name:		kube-addon-manager-minikube
Namespace:	kube-system
Node:		minikube/192.168.99.100
Start Time:	Mon, 28 Nov 2016 12:17:40 +0800
Labels:		component=kube-addon-manager
		version=v5.1
Status:		Pending
IP:		192.168.99.100
Controllers:	<none>
Containers:
  kube-addon-manager:
    Container ID:	
    Image:		gcr.io/google-containers/kube-addon-manager:v5.1
    Image ID:		
    Port:		
    Requests:
      cpu:			5m
      memory:			50Mi
    State:			Waiting
      Reason:			ContainerCreating
    Ready:			False
    Restart Count:		0
    Environment Variables:	<none>
Conditions:
  Type		Status
  Initialized 	True 
  Ready 	False 
  PodScheduled 	True 
Volumes:
  addons:
    Type:	HostPath (bare host directory volume)
    Path:	/etc/kubernetes/
QoS Tier:	Burstable
No events.
```
Then manually download the docker images of 
`gcr.io/google-containers/kube-addon-manager:v5.1`, load it via following command:    

```
$ eval $(minikube-linux-amd64 docker-env)
$ docker load<kubeaddonmanagerv51.tar.bz2 
```
Also the default nginx-3449338310-vna7q is failed, use the same method for manually download
the pause image and load it into the docker system:     

```
$ eval $(minikube-linux-amd64 docker-env)
$ docker load<kubepause30.tar.bz2
```
Also load the dns:    

```
$ eval $(minikube-linux-amd64 docker-env)
$ docker load<kubedns18.tar.bz2
```
### Wordpress Installation
Refers to :    

[https://www.linux-toys.com/?p=887](https://www.linux-toys.com/?p=887)   

Download yaml file:    

```
$ wget https://gist.githubusercontent.com/rusher81572/ddf2e1487b609f294b21a2463a8be104/raw/1ba33c7a2dfbef9118c6043030b76babb0a80c7b/wordpress-k8s -O wordpress.yaml
$ sudo docker pull rusher81572/phpfpm
$ sudo docker pull rusher81572/mysql
$ sudo docker pull rusher81572/nginx
``` 
Create the services from yaml file:    

```
$ kubectl create -f wordpress.yaml
$ minikube-linux-amd64 service nginx --url
http://192.168.99.100:32400
```
Open the url in your browser:    

![/images/2016_11_28_17_24_10_527x489.jpg](/images/2016_11_28_17_24_10_527x489.jpg)    

Manually create the database named `wordpress`:   

```
$ kubectl get pods (To find the Mysql pod name)
$ kubectl exec -it mysql-qe900 bash
$ mysql
$ create database wordpress;
```
Insert the following items in webpage:    

```
Username: root
Password: sql
Database Name: wordpress
Database Host: mysql
```
After installation, now refresh the webpage you will see the installed wordpress.    

### Tips
Login to minikube VM:     

```
$ minikube-linux-amd64 ssh
```

View minikube dashboard URL:    

```
$ minikube-linux-amd64 dashboard --url
http://192.168.99.100:30000
```

View minikube service URL:    

```
$ minikube-linux-amd64 service nginx --url
http://192.168.99.100:32400
```
