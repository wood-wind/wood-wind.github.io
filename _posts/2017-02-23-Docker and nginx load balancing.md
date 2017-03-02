---
layout: post
title: Docker+Nginx+https 负载均衡
date: 2017-03-02
categories: blog
tags: [nginx,docker,https]
description: 在云服务器上利用docker实现简单的负载均衡
---

2017年03月02日22:30:43

云服务器上利用docker建立了三个容器，一个ngnix，两个https

##环境  
* 云服务器一台 CPU:
* 1 vCore
* RAM:768 MB
* Storage:15 GB SSD
* docker 1.13.1
* nginx 

以下为实现过程

###1.拉取https镜像:
``` docker pull https```
###2.docker启动https
```
    docker run -i -t -v 宿主目录:虚拟目录 tomcat /bin/bash  
或：docker run -d -P --name https1 -v /src/webapp:/opt/webapp
```
上面第二条命令会加载主机的 /src/webapp 目录到容器的 /opt/webapp 目录,方便替换文件
如果要进入容器进行交互则:
```
docker run -i -t -v /home/leo/software/:/data/ tomcat /bin/bash 
root@c0bf706ca99c:
```
###3.同样的方法建立https2容器，

###4.建立nginx容器,然后查看容器状态
```
docker run -d -P --name nginx -v /src/webapp:/opt/webapp
```
![状态](http://7xnfbg.com1.z0.glb.clouddn.com/2017-03-02-1.png)


###5.通过映射的端口访问https  

![](http://7xnfbg.com1.z0.glb.clouddn.com/2017-03-02-2.png)

###6.配置nginx,如图

![](http://7xnfbg.com1.z0.glb.clouddn.com/2017-03-02-3.png)

###7.访问测试
不断的刷新页面可以看到在两个https容器切换
![](http://7xnfbg.com1.z0.glb.clouddn.com/2017-03-02-4.png)
![](http://7xnfbg.com1.z0.glb.clouddn.com/2017-03-02-5.png)

注：我已经将其中一个容器的index.html页面修改要不然看不出效果