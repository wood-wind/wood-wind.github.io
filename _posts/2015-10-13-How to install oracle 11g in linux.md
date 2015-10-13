---
layout: post
title: linux 下安装 oracle 11g
date: 2015-10-13
categories: blog
tags: [技术]
description: 最近也在学这个，算是总结吧。
---

2015年10月13日22:46:24

##准备Oracle 12c的安装条件
- 去官方网站下载[http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html) 安装包，有2个包，大约2GB左右

包名称:

	inux_11gR2_database_1of2.zip
	inux_11gR2_database_2of2.zip

- 将安装文件上传

使用rz命令将两个压缩包上传到redhat中
![](http://7xnfbg.com1.z0.glb.clouddn.com/2015-10-13-1.jpg)

- 将压缩包解压

	unzip inux_11gR2_database_1of2.zip
	unzip inux_11gR2_database_2of2.zip

- 得到database文件夹
![](http://7xnfbg.com1.z0.glb.clouddn.com/2015-10-13-2.jpg)

##添加服务器名称(可选)
	eg: 192.168.199.99 	  zjw.localdomain   		  zjw
	<IP-Address>	  <machine-name.domain-name> <machine-name>
	
![](http://7xnfbg.com1.z0.glb.clouddn.com/2015-10-13-3.jpg)

##内核参数调整（不低于以下值）

	vi /etc/sysctl.conf
	
	kernel.shmmni = 4096
	kernel.sem = 250 32000 100 128
	net.ipv4.ip_local_port_range = 9000 65500
	net.core.rmem_default = 262144
	net.core.rmem_max = 4194304
	net.core.wmem_default = 262144
	net.core.wmem_max = 1048576

	加载参数
	
	sysctl -p
	
##调整会话限制

	vi /etc/security/limits.conf
	
最后一行添加
 
	oracle        soft    nproc    8192
	oracle        hard    nproc    16384
	oracle        soft    nofile   32768
	oracle        hard    nofile   65536
	
##安装需要的软件环境

	yum -y install binutils compat-libstdc++ compat-libstdc++-33elfutils-libelf-devel gcc gcc-c++ glibc-devel glibc-headers ksh libaio-devellibstdc++-devel make sysstat unixODBC-devel binutils-* compat-libstdc++*elfutils-libelf* glibc* gcc-* libaio* libgcc* libstdc++* make* sysstat*unixODBC* wget unzip
	yum clean all
	
	rpm -qa|grep binutils(检查binutils安装的版本，未安装为空)
	
##用户环境要求

####创建相关用户、组账号

	#安装组
	groupadd oinstall
	#管理组
	groupadd dba
	groupadd oper
	groupadd asmadmin
	#运行用户
	useradd -g oinstall -G dba,oper,asmadmin oracle
	#设置密码
	passwd oracle
	
####设置SELinux为disabled

	vi /etc/selinux/config
	
![](http://7xnfbg.com1.z0.glb.clouddn.com/2015-10-13-4.jpg)

##安装目录准备
	#创建基本目录
	mkdir -p /oracle/product/11.2.0/db_1
	chown -R oracle:oinstall oracle/
	chmod -R 775 oracle/
	
##用户设置环境信息

	#切换oracle账户
	在oracle用户目录.bash_profile末尾添加
	
	TMP=/tmp; export TMP
	TMPDIR=$TMP; export TMPDIR
	ORACLE_HOSTNAME=zjw.localdomian; export ORACLE_HOSTNAME
	ORACLE_UNQNAME=DB11G; export ORACLE_UNQNAME
	ORACLE_BASE=/oracle; export ORACLE_BASE
	ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1; 
	export ORACLE_HOME
	ORACLE_SID=testdb; export ORACLE_SID
	PATH=/usr/sbin:$PATH; export PATH
	PATH=$ORACLE_HOME/bin:$PATH; export PATH
	LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib; export LD_LIBRARY_PATH
	CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib; export CLASSPATH
	
	#应用配置
	. ./.bash_profile
	
以上为安装oracle前的所有配置修改