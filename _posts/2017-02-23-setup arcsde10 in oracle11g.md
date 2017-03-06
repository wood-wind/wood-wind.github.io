---
layout: post
title: Linux下安装Arcsde10
date: 2017-02-25
categories: blog
tags: [linux,oracle,arcsde]
description:  
---

2017年02月25日 23:26:32

这两天给一个老系统维护，数据中间件还是用的arcsde连接，不是直连的，之前有过在Windows下安装arcsde10，Linux下还是头一回，完成任务后在此记录下。

另外，由于是涉密机器，安装过程记录只好用网上的了。

 ### 1.环境
 
系统：Redhat6.4
oracle: 11g
Arcsge: 10

 ### 2.环境检查
 
检查一下在Linux操作系统下Oracle数据库是否能启动，是否能连通等
```
[oracle@oracledb ~]$ sqlplus / as sysdba  
SQL*Plus: Release 11.2.0.1.0 Production on Wed Feb 22 10:33:47 2012  
Copyright (c) 1982, 2009, Oracle.  All rights reserved.  
Connected to:  
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production  
With the Partitioning, OLAP, Data Mining and Real Application Testing options  
SQL>  
[oracle@oracledb ~]$ sqlplus sys/oracle as sysdba  
SQL*Plus: Release 11.2.0.1.0 Production on Wed Feb 22 10:59:05 2012  
Copyright (c) 1982, 2009, Oracle.  All rights reserved.  
Connected to:  
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production  
With the Partitioning, OLAP, Data Mining and Real Application Testing options  
SQL>  
```
 ### 3.创建arcsde用户
 
我没有创建Arcsde用户，直接用的oracle

创建arcsde用户的命令
```
[root@oracledb ~]# useradd -d /home/arcsde -G oinstall -m arcsde  
[root@oracledb ~]# passwd arcsde  
Changing password for user arcsde.  
New UNIX password:  
BAD PASSWORD: it is based on a dictionary word  
Retype new UNIX password:  
passwd: all authentication tokens updated successfully. 
```
 ### 4.上传安装文件
 
利用FileZilla就行，上传解压后的文件夹，然后赋予读写权限

`[root@oracledb ~]#chmod –R 777 /home/oracle/app/oracle11g_64`

 ### 5.创建SDE的用户、表空间、赋予权限
 
其实创建这些东西不需要我们自己去做的，在oracle11g_64目录下面有一个02.tar的包，解压包有一个createsdeoracle .sql文件：

```
[oracle @oracledb ~]$ ll /home/oracle11g_64/
total 153916
-rwxrwxrwx 1 root root     18699 May 21  2010 01.toc
-rwxrwxrwx 1 root root 157460480 May 21  2010 02.tar
-rwxrwxrwx 1 root root      5366 May 21  2010 b5.nls
-rwxrwxrwx 1 root root      8648 May 21  2010 en.nls
-rwxrwxrwx 1 root root     52923 May 21  2010 install
-rwxrwxrwx 1 root root      9015 May 21  2010 ja.nls
-rwxrwxrwx 1 root root      9011 May 21  2010 jp.nls
-rwxrwxrwx 1 root root     10786 May 21  2010 ko.nls
-rwxrwxrwx 1 root root      7776 May 21  2010 th.nls
-rwxrwxrwx 1 root root      5366 May 21  2010 tw.nls
-rwxrwxrwx 1 root root      5335 May 21  2010 zh.nls
```
这个文件也可以在本地解压出来修改好了再上传上去
主要修改以下几个地方
```
prompt * Connect as system/<password> to create the sde
prompt * tablespace and user.

connect system/<password>

prompt * Create the sde tablespace.
prompt * Before you run this script update LOCATION for the desired datafile pathname.
 
create tablespace sde
datafile '/LOCATION/sde.dbf' size 400M
extent management local uniform size 512K;
```
执行此sql

 ### 6.安装arcsde
```
[root@oracledb ~]#cd /home/oracle11g_64
[root@oracledb oracle11g_64]# ./install -load  
 你可以读懂本行文字吗?  
Is the previous statement legible in your native language? [yes]   
Continue installation with the selected language interaction? [yes]   
如果你已阅读并同意所附许可协议中的条款,请输入'yes'继续安装过程, 否则按<回车>键或输入'no'退出安装过程. [no] yes  
敲回车选择默认项, '?'帮助, '^'返回到  
 上一个问题, 或'q'退出.  
  
 输入CD-ROM mount点: [/mediamnt] /home/oracle/app/oracle11g_64/  
  
 输入安装目录的路径名: [/opt/sde/linux/oracle11g_64] /home/oracle/app/  
ArcSDE version 10.0 for Oracle11g - May 20, 2010    
--------------------------------------------------  
  
ArcSDE Product  
 将要安装的软件模块号: [all]   
  
  
 软件模块选择完毕  
--------------------------  
  
 你选择了安装下列软件模块  
  
ArcSDE Product  
        ArcSDE Server                      
  
 这正确吗? [yes]   
  
 安装时列出文件名吗? [no]   
  
 正在安装软件, 请等待...  

 软件安装完毕  
  
 退出...  
[root@oracledb oracle11g_64]#
```
此处运行./install -road 后可能报错，基本是缺包的问题。我使用挂载本地yum源的方式安装依赖包

 ### 7.修改环境变量
```
[oracle@oracledb ~]$ vim .bash_profile  
# .bash_profile  
  
# Get the aliases and functions  
if [ -f ~/.bashrc ]; then  
        . ~/.bashrc  
fi  
  
# User specific environment and startup programs  
  
export PATH=$PATH:$HOME/bin  
export ORACLE_BASE=/home/oracle/app  
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1
export PATH=$PATH:$ORACLE_HOME/bin  
export ORACLE_SID=orcl  
export TNS_ADMIN=$ORACLE_HOME/network/admin  
export SDEHOME=/home/oracle/app/sdeexe100  
export LD_LIBRARY_PATH=$SDEHOME/lib:/usr/lib:/lib:$ORACLE_HOME/lib  
export PATH=$PATH:$HOME:$ORACLE_HOME/bin:$SDEHOME/bin:/usr/lib64 
```
使之生效

`[oracle@oracledb ~]$ source .bash_profile`

 ### 8.修改配置文件
```
[oracle@oracledb etc]$ vi services.sde  
[oracle@oracledb etc]$ more services.sde  
/* $Id: services.sde,v 1.2 1999/01/22 01:01:35 donna Exp $ */  
#  
# ESRI SDE Remote Protocol  
#   Note:  uncomment the line below to use ESRI's default port  
#  
esri_sde               5151/tcp  
```
在Root用户下/etc/services文件一样添加服务

`[root@oracledb ~]# vi /etc/services `

将sdeexe100设为属于oinstall组
```
[root@oracledb ~]# chown -R oracle:oinstall /home/oracle/app/sdeexe100  
[root@oracledb ~]# chmod -R 755 /home/oracle/app/sdeexe100  
[root@oracledb ~]# su - oracle  
[oracle@oracledb ~]$ cd $SDEHOME  
[oracle@oracledb sdeexe100]$ ll  
总计 20  
drwxr-xr-x 2 oracle oinstall 4096 02-22 13:34 bin  
drwxr-xr-x 2 oracle oinstall 4096 02-22 13:48 etc  
drwxr-xr-x 2 oracle oinstall 4096 02-22 13:34 lib  
drwxr-xr-x 4 oracle oinstall 4096 2010-01-09 locale  
drwxr-xr-x 3 oracle oinstall 4096 02-22 13:34 tools
```
 ### 9.创建SDE的Schema
```
[oracle@oracledb ~]$ sdesetup -o install -d oracle11g -p sde  
  
ESRI ArcSDE Server Setup Utility Mon May 12 17:18:10 2014  
----------------------------------------------------------------  
Install or update ArcSDE, GDB schema objects: Are you sure? (Y/N): y  
  
Checking INSTALL privileges for geodatabase ...  
Current user has privilege to install geodatabase instance.  
  
Checking geodatabase XML datatype support...  
Underlying RDBMS database instance supports XML data type.  
  
Creating ArcSDE schema.....  
Successfully created ArcSDE schema.  
  
Installing St_Geometry ....  
Successfully installed St_Geometry.  
  
Creating geodatabase schema.....  
Successfully created GDB schema.  
  
  
Successfully installed ArcSDE components.  
Refer SDEHOME\etc\sde_setup.log for more details. 
```
 ### 10.更新License
```
[oracle@oracledb ~]$ sdesetup -o update_key -d oracle11g -l /home/arcsde.ecp -u sde -p sde -N 
  
ESRI ArcSDE Server Setup Utility Wed Feb 22 14:41:41 2012  
----------------------------------------------------------------  
Successfully updated authorization key.  
```
 ### 11.启动ArcSDE服务
```
[oracle@oracledb ~]$ sdemon -o start  
Please enter ArcSDE DBA password:
 
 
-------------------------------------------------------
ArcSDE 10.0  for Oracle11g Build 685 Fri May 14 12:05:43  2010
-------------------------------------------------------
 
 
ST_Geometry Schema Owner: (SDE) Type Release: 1007
 
Instance initialized for ((sde)) . . .
 
 
Connected to instance . . .
 
DBMS Connection established...
 
RDBMS:                           "Oracle"
Instance Name:                   "esri_sde"
 
IOMGR Process ID (PID):           62675
 
 
ArcSDE Instance esri_sde started Mon May 12 17:24:07 2014
[oracle@oracledb ~]$
``` 
 
至此整个ArcSDE10.0的安装就完成了，可以去主机用ArcCatalog连接试试了。
