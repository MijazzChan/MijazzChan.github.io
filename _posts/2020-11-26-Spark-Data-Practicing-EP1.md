---
title: Spark Data Practicing-EP1
author: MijazzChan
date: 2020-11-26 01:49:39 +0800
categories: [Data, Hadoop]
tags: [data, hadoop, spark, java]

---

# Spark Data Practicing-EP1

## Testing Hadoop

EP0中给出的`hadoop-3.1.4.tar.gz`

```shell
 mijazz@lenovo  ~/devEnvs  tar -xf ./hadoop-3.1.4.tar.gz
 # 文件结构
 mijazz@lenovo  ~/devEnvs  tree -L 1 ./hadoop-3.1.4 
./hadoop-3.1.4
├── LICENSE.txt
├── NOTICE.txt
├── README.txt
├── bin  # 可执行
├── etc  # 配置
├── include  
├── lib
├── libexec
├── sbin # 可执行
└── share

7 directories, 3 files
```

严格按着官网给出的doc配置

> Unpack the downloaded Hadoop distribution. In the distribution, edit the file etc/hadoop/hadoop-env.sh to define some parameters as follows:
>
> ```shell
>   export JAVA_HOME=/usr/java/latest  # DO NOT ADD THIS LINE
> ```
>
> [Docs](https://hadoop.apache.org/docs/r3.1.4/hadoop-project-dist/hadoop-common/SingleCluster.html) 

也就是让我们在`hadoop`下的配置路径下, 在加上一次JAVA_HOME的路径.

~~严格来讲`~/.bashrc` `~/.zshrc`环境下有就行~~

```shell
echo 'export JAVA_HOME=/home/mijazz/devEnvs/OpenJDK8' >> ./hadoop-3.1.4/etc/hadoop/hadoop-env.sh
```

然后启动一次`hadoop`

```shell
 mijazz@lenovo  ~/devEnvs/hadoop-3.1.4  ./bin/hadoop
Usage: hadoop [OPTIONS] SUBCOMMAND [SUBCOMMAND OPTIONS]
 or    hadoop [OPTIONS] CLASSNAME [CLASSNAME OPTIONS]
  where CLASSNAME is a user-provided Java class

  OPTIONS is none or any of:

--config dir                     Hadoop config directory
--debug                          turn on shell script debug mode
...............
```

## Configure Hadoop

> 官方文档推荐在`etc/hadoop/core-site.xml`和`etc/hadoop/hdfs-site.xml`下各添加一个字段.
>
> 此处不表, 直接贴出上述两个文件.

`core-site.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/mijazz/devEnvs/hadoop-3.1.4/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

`hdfs-site.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-bind-host</name>
        <value>0.0.0.0</value>
    </property>
    <property>
        <name>dfs.namenode.servicerpc-bind-host</name>
        <value>0.0.0.0</value>
    </property>
    <property>
        <name>dfs.namenode.http-bind-host</name>
        <value>0.0.0.0</value>
    </property>
    <property>
        <name>dfs.namenode.https-bind-host</name>
        <value>0.0.0.0</value>
    </property>
</configuration>
```

> 比官网多出来的这几个字段, 方便`interface bind`. 也就是都监听.
>
> 因为之前我尝试在虚拟机`(CentOS8: 192.168.123.5/24, 172.16.0.2/24)`和宿主机`(Windows10: 192.168.123.2/24, 172.16.0.1/24)`和另一台PC`(CentOS7: 192.168.123.4/24, 10.100.0.2/16(OpenVPN-NAT))`之间尝试做分布式. 这几个选项对我的~~恶心人的~~网络拓扑(又是host-only, bridge, openvpn-nat)很有帮助. 官方文档[HDFS Support for Multihomed Networks](https://hadoop.apache.org/docs/r3.1.4/hadoop-project-dist/hadoop-hdfs/HdfsMultihoming.html)

避坑环境变量

> 老样子, 追加`~/.zshrc`和`~/.bashrc`

```shell
export HADOOP_HOME="/home/mijazz/devEnvs/hadoop-3.1.4"
export HADOOP_MAPRED_HOME=$HADOOP_HOME 
export HADOOP_COMMON_HOME=$HADOOP_HOME 
export HADOOP_HDFS_HOME=$HADOOP_HOME 
export YARN_HOME=$HADOOP_HOME 
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native 
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin 
export HADOOP_INSTALL=$HADOOP_HOME 
```

## Configure HDFS

引用一下官方文档两张简洁明了的图

[HDFS Docs](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html)

![HDFS-A](https://hadoop.apache.org/docs/r3.1.4/hadoop-project-dist/hadoop-hdfs/images/hdfsarchitecture.png)

![HDFS](https://hadoop.apache.org/docs/r3.1.4/hadoop-project-dist/hadoop-hdfs/images/hdfsdatanodes.png)

官方文档叙述的架构非常易懂.

这里只提其对用户暴露出的`Shell Commands`接口, 其命令与`Unix/Linux`默认文件系统的接口非常类似.

```shell
 mijazz@lenovo  ~/devEnvs/hadoop-3.1.4  ./bin/hdfs dfs -help
```

即可获得其描述.

继续配置`HDFS`

先对`NameNode`也就是`hdfs`的`master`做一下格式化

```shell
 mijazz@lenovo  ~/devEnvs/hadoop-3.1.4  ./bin/hdfs namenode -format
WARNING: /home/mijazz/devEnvs/hadoop-3.1.4/logs does not exist. Creating.
2020-11-25 23:49:55,071 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = lenovo/127.0.1.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 3.1.4
.............
```

然后同时启动`DataNode`和`NameNode`

```shell
 mijazz@lenovo  ~/devEnvs/hadoop-3.1.4  ./sbin/start-dfs.sh 
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [lenovo]
```

> 这里可以看到`NameNode`和`DataNode`的主机名并不一致, 因为之前配置的ssh登录就是在这里起作用的. 因为`HDFS`主要面向集群, 也就是`NameNode-Master`和`DataNodes-Slaves`大多配置在不同的机器上, 其之间的通信都是通过ssh的. 尽管当前部署是本地单机部署, 他还是会用`ssh`和本地的`sshd`来进行沟通.

然后访问[http://localhost:9870](http://localhost:9870)如果看到hadoop就行.

### 可能会遇到的坑

启动了上述命令, 但是访问`localhost:9870`没反映

那就尝试做一下问题定位

+ 先看看其是不是成功启动了, 用`jps`看看就行.

发现没有`Java`进程在运行.

到`%HADOOP_HOME/logs`下找日志就行

```shell
2020-11-25 23:59:57,076 ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: RECEIVED SIGNAL 1: SIGHUP
2020-11-25 23:59:57,076 ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: RECEIVED SIGNAL 15: SIGTERM
2020-11-25 23:59:57,088 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down DataNode at lenovo/127.0.1.1
************************************************************/
```

看到了一些奇怪的东西`127.0.1.1`. 该指向在默认的`hosts`中存在, 具体原因不表, 具体情况因`Linux`发行版而异.

看到这里, 很明显是主机名被解析到了一个不能访问到本机的ip上, 导致是`DataNode`和`NameNode`之前没心跳.

```shell
 mijazz@lenovo  ~/devEnvs/hadoop-3.1.4/logs  cat /etc/hosts
# Host addresses
127.0.0.1  localhost
127.0.1.1  lenovo     # Here
.............
```

简单修改就行.

```shell
 mijazz@lenovo  ~  cat /etc/hosts
# Host addresses
127.0.0.1  localhost
127.0.0.1  lenovo     # 127.0.0.1
............
```

+ 还是启动不了, 尝试一下用`hadoop namenode`和`hadoop datanode`放在`shell`里启动. 如果此时`jps`能看到活的进程, 并且`curl localhost:9870`有返回, 后续可以尝试:

  > 有条件可以去读一下几个启动脚本的源码, 如果启动脚本拉起不了, 既有可能是目录权限问题或者用户权限问题.
  >
  > 因为官方本就推荐hadoop运行于一个独立的用户/用户组.
  
  ```shell
  stop-all.sh
  hadoop-daemon.sh start namenode
hadoop-daemon.sh start datanode
  ```
  





## Configure YARN

> YARN = Yet Another Resource Negotiator
>
> 什么是YARN -> [YARN Architechture](https://hadoop.apache.org/docs/r3.1.4/hadoop-yarn/hadoop-yarn-site/YARN.html)

![YARN](https://hadoop.apache.org/docs/r3.1.4/hadoop-yarn/hadoop-yarn-site/yarn_architecture.gif)

继续贴我自己使用的配置文件

`etc/hadoop/mapred-site.xml`

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

`etc/hadoop/yarn-site.xml`

```xml
<?xml version="1.0"?>

<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
    <property>
        <name>yarn.webapp.ui2.enable</name>
        <value>true</value>
    </property>
</configuration>

```

直接用命令拉起`NodeManager`和`ResourceManager`即可.

```shell
 mijazz@lenovo  ~/devEnvs/hadoop-3.1.4  start-yarn.sh
```

### 可能的坑

用该命令拉起尝试即可

```shell
yarn-daemon.sh start resourcemanager
yarn-daemon.sh start nodemanager
yarn-daemon.sh start historyserver
```

另外定位`%HADOOP_HOME/logs`下即可.



## ~~有个隐藏的坑~~

随后再更新



## Minor Adjustment

如果报`Unable to load native-hadoop library for your platform`

试着执行

```shell
 mijazz@lenovo  ~/devEnvs  hadoop checknative
```

如果下列是一堆`no`, 不要上百度找答案, 百度上还有说不带`lib`的.

只需要在`etc/hadoop/hadoop-env.sh`追加一个`JVM`参数就行.

```shell
export HADOOP_OPTS="${HADOOP_OPTS} -Djava.library.path=${HADOOP_HOME}/lib/native"
```

随后

```shell
 mijazz@lenovo  ~/devEnvs/spark-2.4.7  hadoop checknative
2020-11-27 17:49:39,805 INFO bzip2.Bzip2Factory: Successfully loaded & initialized native-bzip2 library system-native
2020-11-27 17:49:39,809 INFO zlib.ZlibFactory: Successfully loaded & initialized native-zlib library
2020-11-27 17:49:39,817 WARN erasurecode.ErasureCodeNative: ISA-L support is not available in your platform... using builtin-java codec where applicable
2020-11-27 17:49:39,864 INFO nativeio.NativeIO: The native code was built without PMDK support.
Native library checking:
hadoop:  true /home/mijazz/devEnvs/hadoop-3.1.4/lib/native/libhadoop.so.1.0.0
zlib:    true /usr/lib/libz.so.1
zstd  :  true /usr/lib/libzstd.so.1
snappy:  true /usr/lib/libsnappy.so.1
lz4:     true revision:10301
bzip2:   true /usr/lib/libbz2.so.1
openssl: false EVP_CIPHER_CTX_cleanup
ISA-L:   false libhadoop was built without ISA-L support
PMDK:    false The native code was built without PMDK support.
```

再次运行时警告就会消失.

## 验证

![Hadoop Index Page](/assets/img/blog/20201127/Screenshot_20201127_171715.png)

+ http://your.host.or.ip:9870

+ http://you.host.or.ip:8088

+ `jps`

  ```shell
   mijazz@lenovo  ~/devEnvs/hadoop-3.1.4  jps
  2160 NameNode
  7297 Jps
  6084 ApplicationHistoryServer
  2551 ResourceManager
  2413 NodeManager
  2237 DataNode
  ```

  