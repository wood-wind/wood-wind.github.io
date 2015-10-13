---
layout: post
title: RedHat9下搭建Web应用环境(上)
date: 2015-10-11
categories: blog
tags: [技术]
description: 最近在学这个，算是总结吧。
---

2015年10月11日22:57:16

最近在学linux配置Java环境，将项目部署到linux下，并能够运行。

##获取工具包
- VMware(虚拟机)

- redhat9

- [jdk-8u60-linux-i586.tar.gz[点此下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

- tomcat7.0.64[点此下载](http://tomcat.apache.org/download-70.cgi)

- Mysql

##VMware的安装（略）

##redhat9的安装

如果系统包分了几部分可能会出现**换光盘**的情形。
	![](http://7xnfbg.com1.z0.glb.clouddn.com/2015-10-11-1.jpg)
点击右下角的小光盘图标
	![](http://7xnfbg.com1.z0.glb.clouddn.com/2015-10-11-2.jpg)
以及选择第二个文件
	![](http://7xnfbg.com1.z0.glb.clouddn.com/2015-10-11-3.jpg)
完成之后如果直接点确定发现还是不行的，得再点击小光盘图标然后选**连接**才行

##网络设置

####修改IP地址

	vi /etc/sysconfig/network-scripts/ifcfg-eth0
	DEVICE=eth0				设置网络接口名称
	ONBOOT=yes					设置网络接口在系统启动时激活。
	BOOTPROTO=static			配置为静态地址
	IPADDR=192.168.199.99		IP地址
	NETMASK=255.255.255.0		子网掩码
	GATEWAY=192.168.199.1		网关
	DNS1=8.8.8.8
	DNS2=8.8.4.4

也可在下面设置

####修改网关

	vi /etc/sysconfig/network
	NETWORKING=yes
	HOSTNAME=wu
	GATEWAY=192.168.199.1

####修改DNS

	vi /etc/resolv.conf
	DNS1=8.8.8.8
	DNS2=8.8.4.4

####重新启动网络配置

	service network restart
	
####最后Ping www.baidu.com 看是否能Ping通
	
##安装JDK

####法一:直接用yum安装lrzsz（推荐）

	yum install lrzsz -y
	
安装完成之后  
使用rz(上传)  
同样  
	sz filename(下载)
	
####法二:通过FlashFXP工具(或者其他工具)

将下载到电脑上的JDK用工具传到redhat某文件夹下
![](http://7xnfbg.com1.z0.glb.clouddn.com/2015-10-12-1.jpg)

	
##JDK环境配置

####解压 jdk.tar.gz 包

	tar -zxvf jdk-8u60-linux-i586.tar.gz	(解压文件)
	mv jdk-8u60-linux-i586.tar.gz /usr/local/   (将JDK拷贝到目标路径下)

####设置环境变量

	#编辑环境变量配置文件 
	vi /etc/profile

在文件开头或末尾添加  
	JAVA_HOME=/usr/local/java/jdk(安装的版本号/文件夹名) 
	CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
	PATH=$PATH:$JAVA_HOME/bin 
	
保存退出，并执行命令： 
	
	#执行 
	source /etc/profile 

####检测  
输入java -version查看是否配置正确,
也可以写个"Hello World"进行测试

	#新建一个Test.java文件，在其中输入以下代码并保存： 
	public class test { 
		public static void main(String args[]) { 
			System.out.println("Hello Worle!"+args[0]); 
		} 
	} 
	
编译：在终端执行命令 javac Test.java  
运行：在终端执行命令 java Test wu  
当下方出现“Hello Worle!wu”字样则jdk运行正常。	
	
##Tomcat的安装
上传方法同 jdk 一样,rz 或者 wget 命令把文件移动到/etc/local/目录下(或直接上传至该目录)  
	#解压tomcat到当前目录 
	tar –xvzf apache-tomcat-7.0.64.tar.gz  
即完成了tomcat的部署

##启动tomcat服务
将目录切换到opt/tomcat/bin执行  
	./startup.sh  
提示服务开启

##在客户端访问tomcat首页
**http://localhost:8080/**如果正常显示tomcat首页表明配置ok。
![](http://7xnfbg.com1.z0.glb.clouddn.com/2015-10-12-2.jpg)

##关闭防火墙
如果部署完成但是依然不能正常显示tomcat首页可能是未关防火墙  
	#关闭系统防火墙
	service iptables stop/start

##部署一个web项目
将文件为test.war的包上传到opt/tomcat/webapps目录下，然后重启tomcat  
在客户端输入http://localhost:8080/ssh2，显示登录页面表明项目部署成功。

