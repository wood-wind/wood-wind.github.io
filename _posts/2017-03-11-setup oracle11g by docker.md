---
layout: post
title: 利用docker安装oracle11g
date: 2017-03-12
categories: blog
tags: [oracle,docker]
description: 并不是一键执行的xe oracle，是企业版的
---

2017年03月12日23:03:22

通过这种方法安装能够极大的简化安装步骤，非常方便。暂不清楚性能方面的差异

## 环境  
* 系统:Ubuntu 16.04 64位
* docker 1.13.1
* oracle 11g 

以下为实现过程

### 1.安装前准备:

在官网下载安装文件，一共两个文件  
[http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index-092322.html](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index-092322.html)

docker自动化脚本安装  
` curl -sSL https://get.docker.com/ | sh `


### 2.文件上传到服务器

 * 将下载好的安装文件上传到要安装oracle的服务器上。
 * 安装文件放在install_folder文件夹下
 * 同时将安装文件解压
 
 可以通过Filezilla上传  
![](https://lh3.google.com/u/0/d/0B8KZYo2iZmoGRzd3MEFEUFVNbWc=w1366-h638-iv1)

### 3.容器运行

` docker run --privileged=true --name oracle11g -p 1521:1521 -v /home/install_folder/:/install jaspeen/oracle-11g`

 * --privileged=true    Docker将拥有访问host所有设备的权限
 * --name oracle11g    容器名称
 * -p 1521:1521       访问端口
 * -v /home/install_folder/:/install   文件夹映射，左为本机右为容器
 * jaspeen/oracle-11g    运行的镜像

### 4.等待安装完成

![](https://lh3.google.com/u/0/d/0B8KZYo2iZmoGNktkYUphc29aS3M=w1366-h638-iv1)


### 5.连接测试  

 * hostname: localhost 或 IP
 * port: 1521
 * sid: orcl
 * username: sys
 * password: oracle

### 6.后续

1.暂不清楚如何自定义参数，尝试docker run命令加参数并不能成功执行  
2.此方法安装完成后需要修改字符编码

### 7.字符编码的修改
```
$ sqlplus / as sysdba
SQL> shutdown immediate;
SQL> startup mount;
SQL> ALTER SYSTEM ENABLE RESTRICTED SESSION;
SQL> ALTER SYSTEM SET JOB_QUEUE_PROCESSES=0;
SQL> ALTER SYSTEM SET AQ_TM_PROCESSES=0;
SQL> alter database open;
SQL>ALTER DATABASE CHARACTER SET INTERNAL_USE ZHS16GBK; //跳过超子集检测
SQL> shutdown immediate;
SQL> startup;
SQL> exit;
```