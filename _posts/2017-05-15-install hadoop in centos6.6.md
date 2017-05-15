---
layout: post
title: CentOS6下配置Hadoop 2.7.3集群
date: 2017-5-15
categories: blog
tags: [hadoop,centos]
description:  
---

2017年5月15日 20:28:53

这周机房四台服务器要我格了装CentOS，做大数据分析，本文记录以Hadoop YARN和Spark为基础来构建大数据平台的过程。

实验机器为google cloud的3台实例
### 环境
* CentOS 6.6
* Hadoop 2.7.3
* Java 1.8

### 1.基础准备
 1. 配置java环境 （略）
 2. 配置SSH免密登陆
 3. hosts文件映射
 
| 主机名        | IP           |
| ------------- |:------------:|
| instance-1    | 10.140.0.2   |
| instance-2    | 10.140.0.3   |
| instance-3    | 10.140.0.4   |

直接修改/etc/hosts文件即可
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.140.0.2      instance-1
10.140.0.3 instance-2.c.northern-card-162408.internal instance-2  # Added by Google
10.140.0.4      instance-3
169.254.169.254 metadata.google.internal  # Added by Google
```
修改主机在
`/etc/sysconfig/network`
文件中，HOSTNAME = [主机名]

修改主机名需要重启

### 2.配置Hadoop  
这里将hadoop安装在`/home/app/hadoop`目录，下面的操作非特殊说明都是在instance-1上进行操作，操作用户为silen的普通用户。
 **1.下载hadoop 2.7.3**
 
`wget http://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz`

或者通过Filezilla上传

**解压并进入到hadoop目录**

```
tar -zxvf hadoop-2.7.3.tar.gz
mv hadoop-2.7.3 /usr/hadoop
cd /usr/hadoop
```

将以下环境变量添加到自己的~/.bash_profile或者/etc/profile中

```
export HADOOP_HOME=/home/app/hadoop
PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
CLASSPATH=$CLASSPATH:$HADOOP_HOME/share/hadoop/common/hadoop-common-2.7.3.jar:$HADOOP_HOME/share/hadoop/common/lib/commons-cli-1.2.jar:$HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.7.3.jar
export HADOOP_PREFIX=/home/app/hadoop
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HADOOP_HOME/lib/native/
PATH=$PATH:$HOME/bin
export PATH
export CLASSPATH 
```
PATH变量中最好不要添加sbin目录，因为会与spark的sbin目录下的脚本冲突，需要启动的时候直接手动到相应目录下启动相应服务

scp复制到其他机器：

```
scp ~/.bash_profile instance-2:~
scp ~/.bash_profile instance-2:~
```

**所有修改的设置都可以通过在instance-1下配置好后复制到其他节点**

分别在三台机器上执行source使环境变量生效：

`source ~/.bashrc`

如果环境变量配置都没有问题的话，现在已经可以查看到hadoop的版本等信息了：

`hadoop version`

 **2.验证Hadoop单机配置**
 
修改`/home/app/hadoop`目录下的`etc/hadoop/hadoop-env.sh`文件，修改Java路径：

`export JAVA_HOME=/home/app/java`

通过拷贝hadoop的配置文件，并在调用hadoop自带示例中的正则表达式来搜索配置文件，并将结果输出到output：
```
cd /home/app/hadoop
cp etc/hadoop/*.xml input
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep input output 'dfs[a-z.]+'
cat output/*
```
如果执行中没有报错，并且输出如下，则表示hadoop环境变量配置完成：

`dfsadmin`

 **3.开始配置集群环境**

修改Hadoop配置文件

下面的操作都是在hadoop目录之中的`etc/hadoop`路径下，即`/home/app/hadoop/etc/hadoop`，该文件夹存放了hadoop所需要的几乎所有配置文件。
需要修改的配置文件主要有：  
hadoop-env.sh, core-site.xml, hdfs-site.xml, mapred-site.xml, yarn-env.sh, yarn-site.xml, slaves

 * **hadoop-env.sh**
 
除了上面需要在其中添加JAVA_HOME之外，还需要增加HADOOP_PREFIX变量：

`export HADOOP_PREFIX=/home/app/hadoop`

 * **core-site.xml**
 
```
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://instance-1:9000</value>
    <description>HDFS的URL，文件系统：//namenode标识:端口号</description>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/home/app/hadoop/tmp</value>
    <description>namenode上本地的hadoop临时文件夹</description>
  </property>
</configuration>
```

这里指定instance-1为namenode及相应端口号，并设置本地的临时文件夹为hadoop安装目录下的tmp，该目录需要手动创建

 * **hdfs-site.xml**
 
```
<configuration>
  <property>
    <name>dfs.name.dir</name>
    <value>/home/app/hadoop/hdfs/name</value>
    <description>namenode上存储hdfs名字空间元数据</description>
  </property>
  <property>
    <name>dfs.data.dir</name>
    <value>/home/app/hadoop/hdfs/data</value>
    <description>datanode上数据块的物理存储位置</description>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>3</value>
    <description>副本个数，配置默认是3，应小于datanode机器数量</description>
  </property>
</configuration>
```

指定namenode和datanode数据的存储位置（需要手动创建），以及副本个数

 * **mapred-site.xml**
 
```
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>instance-1:10020</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>instance-1:19888</value>
  </property>
</configuration>
```

* **yarn-env.sh**

```
# export JAVA_HOME=/home/y/libexec/jdk1.6.0/
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
```

修改JAVA_HOME

 * **yarn-site.xml**
 
```
<configuration>
<!-- Site specific YARN configuration properties -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>instance-1</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>instance-1:8099</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>instance-1:8032</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>instance-1:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>instance-1:8031</value>
  </property>
</configuration>
```

指定resourcemanager为instance-1，并修改相应端口，这些端口如果不修改都有默认值，可以根据自己的网络情况进行修改

 * **slaves**
 
```
instance-1
instance-2
instance-3
```

instance-1即是namenode，同时也是datanode

创建刚才配置中用到的文件夹：

`mkdir -p /home/app/hadoop/hdfs/data /home/app/hadoop/hdfs/name /usr/hadoop/tmp`

配置完成之后将该配置复制到instance-2和instance-3上：

```
scp -r /usr/hadoop slave1:/usr/
scp -r /usr/hadoop slave2:/usr/
```

### 3.启动Hadoop集群

 **1.启动hdfs**
 
记得其他2个slave上的环境变量都已经生效
在instance-1上格式化namenode：

`hdfs namenode -format`

在instance-1上执行以下命令启动hadoop：

`start-dfs.sh`

启动完成之后在master上启动jps命令查看其java进程：

```
3226 NameNode
5334 Jps
3508 SecondaryNameNode
3352 DataNode
```

查看instance-2和instance-3上进程：

instance-2：

```
1959 DataNode
2720 Jps
```

instance-3:

```
2704 Jps
1963 DataNode
```

找一个能够访问instance-1的浏览器通过50070端口可以查看namenode和datanode情况

`http://[instance-1_IP]:50070`

如果看到Summary中的Live Nodes显示为3，并且Configured Capacity中显示的DFS总大小刚好为三台机器的可用空间大小，则表示已经配置没有问题

### 4.启动yarn

执行以下命令启动yarn：

`start-yarn.sh`

使用jps查看master及2个slave：
instance-1：

```
3667 ResourceManager
3226 NameNode
3508 SecondaryNameNode
5334 Jps
3352 DataNode
3767 NodeManager
```

instance-2：

```
1959 DataNode
2062 NodeManager
2720 Jps
```

instance-3：

```
2704 Jps
1963 DataNode
2065 NodeManager
```

根据我们的配置，通过instance-1的端口8099端口可以在web端查看集群的内存、CPU、及任务调度情况

`http://192.168.1.187:8099/cluster`

如果显示的Memory Total、Active Nodes等内容与你的实际相符，则表示yarn启动成功
通过各节点的8042端口可以查看各节点的资源情况，如查看slave1的节点信息：

`http://192.168.1.188:8042`

也可以通过以下命令查看hdfs的全局信息：

`[silen@instance-1 spark]$ hdfs dfsadmin -report`

输出：

```
Configured Capacity: 62987169792 (58.66 GB)
Present Capacity: 51942948864 (48.38 GB)
DFS Remaining: 51942862848 (48.38 GB)
DFS Used: 86016 (84 KB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0

-------------------------------------------------
Live datanodes (3):

Name: 10.140.0.3:50010 (instance-2)
Hostname: instance-2.c.northern-card-162408.internal
Decommission Status : Normal
Configured Capacity: 20995723264 (19.55 GB)
DFS Used: 28672 (28 KB)
Non DFS Used: 3493302272 (3.25 GB)
DFS Remaining: 17502392320 (16.30 GB)
DFS Used%: 0.00%
DFS Remaining%: 83.36%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon May 15 08:06:52 UTC 2017


Name: 10.140.0.2:50010 (instance-1.c.northern-card-162408.internal)
Hostname: instance-1.c.northern-card-162408.internal
Decommission Status : Normal
Configured Capacity: 20995723264 (19.55 GB)
DFS Used: 28672 (28 KB)
Non DFS Used: 4073074688 (3.79 GB)
DFS Remaining: 16922619904 (15.76 GB)
DFS Used%: 0.00%
DFS Remaining%: 80.60%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon May 15 08:06:52 UTC 2017


Name: 10.140.0.4:50010 (instance-3)
Hostname: instance-3.c.northern-card-162408.internal
Decommission Status : Normal
Configured Capacity: 20995723264 (19.55 GB)
DFS Used: 28672 (28 KB)
Non DFS Used: 3477843968 (3.24 GB)
DFS Remaining: 17517850624 (16.31 GB)
DFS Used%: 0.00%
DFS Remaining%: 83.44%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon May 15 08:06:52 UTC 2017
```

### 5. 启动Job History

`mr-jobhistory-daemon.sh start historyserver`

instance-1上的jps进程：

```
3667 ResourceManager
3508 SecondaryNameNode
4135 JobHistoryServer
3767 NodeManager
3352 DataNode
5448 Jps
3226 NameNode
```

查看web页面：http://[instance-1_IP]:19888

至此整个Hadoop集群已经启动成功
Job history并不是必须的