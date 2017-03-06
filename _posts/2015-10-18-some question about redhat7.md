---
layout: post
title: RedHat7 问题总结
date: 2015-10-18
categories: blog
tags: [技术]
description: 最近在学这个，算是总结吧。
---

2015年10月18日22:47:33

在安装oracle的过程中遇到了非常非常多的问题，我将遇到的问题以标题列出，并给出我的解决方法以及方法出处。

### 关于网络配置

[网络配置问题，将IP地址设为静态  以及手动分区](http://www.osyunwei.com/archives/7702.html)

### 关于yum的问题

[连接网络后使用yum install -y lrzsz报no package lrzsz available错误](http://www.cnblogs.com/kofxxf/p/3658610.html)

解决方案：

使用手动方式安装lrzsz
1. 下载lrzsz包

	[http://down1.chinaunix.net/distfiles/lrzsz-0.12.20.tar.gz](http://down1.chinaunix.net/distfiles/lrzsz-0.12.20.tar.gz)
2. 解压文件

	tar zxvf lrzsz-1.12.20.tar.gz
	
3. 进入解压目录执行命令

	./configure --prefix=/usr/local/lrzsz
	make
	make install
	
4. 建立软链接

	cd /usr/bin
	ln -s /usr/local/lrzsz/bin/lrz rz
	ln -s /usr/local/lrzsz/bin/lsz sz
	
5. 测试

使用rz命令弹出SecureCRT上传窗口

### yum命令总提示"没有可用软件包"

尝试用挂载iso镜像然后利用iso作为yum源安装软件

上传ios错误。。。睡觉
