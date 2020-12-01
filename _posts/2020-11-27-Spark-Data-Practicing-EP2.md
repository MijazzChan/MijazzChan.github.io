---
title: Spark Data Practicing-EP2
author: MijazzChan
date: 2020-11-27 17:55:17 +0800
categories: [Data, Spark]
tags: [data, spark, java]
---

# Spark Data Practicing-EP2

## Testing Spark

EP0中的spark

```shell
 mijazz@lenovo  ~/devEnvs  ll -a
total 161M
drwxr-xr-x  8 mijazz mijazz 4.0K Nov 27 17:27 .
drwx------ 38 mijazz mijazz 4.0K Nov 27 17:27 ..
drwxr-xr-x  8 mijazz mijazz 4.0K Nov  9 20:23 OpenJDK8
drwxr-xr-x 11 mijazz mijazz 4.0K Nov 25 23:49 hadoop-3.1.4
-rw-r--r--  1 mijazz mijazz 161M Nov 24 16:49 spark-2.4.7-bin-without-hadoop.tgz
 mijazz@lenovo  ~/devEnvs  tar -xf spark-2.4.7-bin-without-hadoop.tgz 
 mijazz@lenovo  ~/devEnvs  mv ./spark-2.4.7-bin-without-hadoop ./spark-2.4.7                   
 mijazz@lenovo  ~/devEnvs  tree -L 1 ./spark-2.4.7 
./spark-2.4.7
├── LICENSE
├── NOTICE
├── R
├── README.md
├── RELEASE
├── bin
├── conf
├── data
├── examples
├── jars
├── kubernetes
├── licenses
├── python
├── sbin
└── yarn

11 directories, 4 files
```

## Configure Spark

老样子, 变量`~/.zshrc`, `~/.bashrc`

```shell
# Spark Environment Variable
export SPARK_HOME="/home/mijazz/devEnvs/spark-2.4.7"
export PATH="$PATH:${SPARK_HOME}/bin:${SPARK_HOME}/sbin"
export HADOOP_CONF_DIR="${HADOOP_HOME}/etc/hadoop"
export SPARK_DIST_CLASSPATH="$(hadoop classpath)"
```

`spark`下也有`conf`, 看一眼

```shell
 mijazz@lenovo  ~/devEnvs/spark-2.4.7  tree ./conf
./conf
├── docker.properties.template
├── fairscheduler.xml.template
├── log4j.properties.template
├── metrics.properties.template
├── slaves.template
├── spark-defaults.conf.template
└── spark-env.sh.template

0 directories, 7 files
```

这些`template`里面都写着`spark`的默认配置.

```
 mijazz@lenovo  ~/devEnvs/spark-2.4.7  cat ./conf/spark-defaults.conf.template 
# ...

# Example:
# spark.master                     spark://master:7077
# spark.eventLog.enabled           true
# spark.eventLog.dir               hdfs://namenode:8021/directory
# spark.serializer                 org.apache.spark.serializer.KryoSerializer
# spark.driver.memory              5g
# spark.executor.extraJavaOptions  -XX:+PrintGCDetails -Dkey=value -Dnumbers="one two three"
```

但是默认的`spark`是有`pre-built with hadoop`的. 这次我是采用的自己的`hadoop`分离开搭建, 以便`spark`的`RDD`和`hadoop`的`MapReduce`我都能分开用.

所以这次的`spark`的运行模式是`yarn` -> [`YARN on Hadoop`](https://spark.apache.org/docs/2.4.7/running-on-yarn.html), 所以`spark.master`字段要改`yarn`

**直接上配置**

`spark-defaults.conf`

```shell
spark.master  yarn
spark.eventLog.enabled  true
# 如果你在定义hadoop的hdfs时采用了自定义端口, 在这里更改
spark.eventLog.dir hdfs://localhost:9000/tmp/spark-logs
spark.history.provider org.apache.spark.deploy.history.FsHistoryProvider
spark.history.fs.logDirectory hdfs://localhost:9000/tmp/spark-logs
spark.history.fs.update.interval 10s
spark.history.ui.port 18080
```

历史记录应该是可以记录在本地的, 但是为了方便, 此处将其一共上传至`hdfs`, 方便追溯`job history`.

```
start-history-server.sh
```

### 可能遇到的坑

> `hdfs`里的`/tmp`权限默认应该是可写的, 但是有可能在拉起记录进程的时候, 他访问文件夹的时候, 空的时候它不去创建.

```shell
 ✘ mijazz@lenovo  ~/devEnvs/spark-2.4.7  start-history-server.sh
starting org.apache.spark.deploy.history.HistoryServer, logging to /home/mijazz/devEnvs/spark-2.4.7/logs/spark-mijazz-org.apache.spark.deploy.history.HistoryServer-1-lenovo.out
failed to launch: nice -n 0 /home/mijazz/devEnvs/spark-2.4.7/bin/spark-class org.apache.spark.deploy.history.HistoryServer
        at org.apache.spark.deploy.history.FsHistoryProvider.<init>(FsHistoryProvider.scala:207)
        at org.apache.spark.deploy.history.FsHistoryProvider.<init>(FsHistoryProvider.scala:86)
        ... 6 more
  Caused by: java.io.FileNotFoundException: File does not exist: hdfs://localhost:9000/spark-logs
        at org.apache.hadoop.hdfs.DistributedFileSystem$29.doCall(DistributedFileSystem.java:1586)
        at org.apache.hadoop.hdfs.DistributedFileSystem$29.doCall(DistributedFileSystem.java:1579)
        at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
        at org.apache.hadoop.hdfs.DistributedFileSystem.getFileStatus(DistributedFileSystem.java:1594)
        at org.apache.spark.deploy.history.FsHistoryProvider.org$apache$spark$deploy$history$FsHistoryProvider$$startPolling(FsHistoryProvider.scala:257)
        ... 9 more
```

用`hdfs dfs -mkdir /tmp/spark-logs`再`hdfs dfs -ls /tmp`确认一下`7xx`权限即可.

## Running Spark

> [spark-submit](https://spark.apache.org/docs/2.4.7/submitting-applications.html)
>
> [cluster](https://spark.apache.org/docs/2.4.7/cluster-overview.html) client模式, cluster模式.

先理解一下spark的运行结构

![](https://spark.apache.org/docs/2.4.7/img/cluster-overview.png)

测试一下http://your.ip.or.host:18080

查看`history server`能不能拉起. 因为其也是作为`yarn`运行`hadoop`上的, 所以此处的`ip`应该是`master`的.

按照包里给出的`example jar`, 用`spark-submit`来提交jar包运行.

```shell
spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode client $SPARK_HOME/examples/jars/spark-examples_2.11-2.4.7.jar 10
```

至于`cluster`模式和`client`模式, 主要看`spark driver`运行在哪一侧. 如果是`cluster`模式, 在该次作业中, `spark`会把driver也交给`yarn master`来运行.

```
2020-11-29 15:56:51,224 INFO scheduler.DAGScheduler: Job 0 finished: reduce at SparkPi.scala:38, took 0.920297 s
Pi is roughly 3.1402631402631402
```

如果成功, 可以看到有该行输出. 记得`| grep "Pi is roughly"`.

同时也将会在`Spark History Server`即18080端口, 和`Yarn Cluster`即8088端口看见`yarn spark`的运行记录以及`event logs`.



### 参考运行

```shell
 mijazz@lenovo  ~/devEnvs/spark-2.4.7  spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode client  $SPARK_HOME/examples/jars/spark-examples_2.11-2.4.7.jar 10  
2020-11-29 15:56:38,173 WARN util.Utils: Your hostname, lenovo resolves to a loopback address: 127.0.0.1; using 192.168.123.2 instead (on interface enp4s0)
2020-11-29 15:56:38,173 WARN util.Utils: Set SPARK_LOCAL_IP if you need to bind to another address
2020-11-29 15:56:38,404 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
2020-11-29 15:56:38,541 INFO spark.SparkContext: Running Spark version 2.4.7
2020-11-29 15:56:38,556 INFO spark.SparkContext: Submitted application: Spark Pi
2020-11-29 15:56:38,593 INFO spark.SecurityManager: Changing view acls to: mijazz
2020-11-29 15:56:38,593 INFO spark.SecurityManager: Changing modify acls to: mijazz
2020-11-29 15:56:38,593 INFO spark.SecurityManager: Changing view acls groups to: 
2020-11-29 15:56:38,593 INFO spark.SecurityManager: Changing modify acls groups to: 
2020-11-29 15:56:38,594 INFO spark.SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(mijazz); groups with view permissions: Set(); users  with modify permissions: Set(mijazz); groups with modify permissions: Set()
2020-11-29 15:56:38,771 INFO util.Utils: Successfully started service 'sparkDriver' on port 39113.
2020-11-29 15:56:38,790 INFO spark.SparkEnv: Registering MapOutputTracker
2020-11-29 15:56:38,802 INFO spark.SparkEnv: Registering BlockManagerMaster
2020-11-29 15:56:38,804 INFO storage.BlockManagerMasterEndpoint: Using org.apache.spark.storage.DefaultTopologyMapper for getting topology information
2020-11-29 15:56:38,805 INFO storage.BlockManagerMasterEndpoint: BlockManagerMasterEndpoint up
2020-11-29 15:56:38,811 INFO storage.DiskBlockManager: Created local directory at /tmp/blockmgr-01fd513c-7e08-401b-b6ea-46a0a268accf
2020-11-29 15:56:38,823 INFO memory.MemoryStore: MemoryStore started with capacity 366.3 MB
2020-11-29 15:56:38,857 INFO spark.SparkEnv: Registering OutputCommitCoordinator
2020-11-29 15:56:38,915 INFO util.log: Logging initialized @1320ms
2020-11-29 15:56:38,955 INFO server.Server: jetty-9.3.z-SNAPSHOT, build timestamp: 2019-08-14T05:28:18+08:00, git hash: 84700530e645e812b336747464d6fbbf370c9a20
2020-11-29 15:56:38,972 INFO server.Server: Started @1378ms
2020-11-29 15:56:38,989 INFO server.AbstractConnector: Started ServerConnector@33aa93c{HTTP/1.1,[http/1.1]}{0.0.0.0:4040}
2020-11-29 15:56:38,989 INFO util.Utils: Successfully started service 'SparkUI' on port 4040.
2020-11-29 15:56:39,006 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@6bd51ed8{/jobs,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,007 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@3b65e559{/jobs/json,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,007 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@bae47a0{/jobs/job,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,009 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@1c05a54d{/jobs/job/json,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,010 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@65ef722a{/stages,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,010 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@5fd9b663{/stages/json,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,011 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@214894fc{/stages/stage,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,012 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@1c4ee95c{/stages/stage/json,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,012 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@79c4715d{/stages/pool,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,013 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@5aa360ea{/stages/pool/json,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,013 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@6548bb7d{/storage,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,013 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@e27ba81{/storage/json,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,014 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@54336c81{/storage/rdd,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,014 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@1556f2dd{/storage/rdd/json,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,015 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@35e52059{/environment,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,015 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@62577d6{/environment/json,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,016 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@49bd54f7{/executors,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,016 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@6b5f8707{/executors/json,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,017 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@772485dd{/executors/threadDump,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,017 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@5a12c728{/executors/threadDump/json,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,022 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@79ab3a71{/static,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,023 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@5a772895{/,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,024 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@39fc6b2c{/api,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,024 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@7cc9ce8{/jobs/job/kill,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,025 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@2e27d72f{/stages/stage/kill,null,AVAILABLE,@Spark}
2020-11-29 15:56:39,026 INFO ui.SparkUI: Bound SparkUI to 0.0.0.0, and started at http://lenovo.lan:4040
2020-11-29 15:56:39,035 INFO spark.SparkContext: Added JAR file:/home/mijazz/devEnvs/spark-2.4.7/examples/jars/spark-examples_2.11-2.4.7.jar at spark://lenovo.lan:39113/jars/spark-examples_2.11-2.4.7.jar with timestamp 1606636599035
2020-11-29 15:56:39,634 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
2020-11-29 15:56:39,880 INFO yarn.Client: Requesting a new application from cluster with 1 NodeManagers
2020-11-29 15:56:39,933 INFO conf.Configuration: resource-types.xml not found
2020-11-29 15:56:39,933 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2020-11-29 15:56:39,945 INFO yarn.Client: Verifying our application has not requested more than the maximum memory capability of the cluster (8192 MB per container)
2020-11-29 15:56:39,946 INFO yarn.Client: Will allocate AM container, with 896 MB memory including 384 MB overhead
2020-11-29 15:56:39,946 INFO yarn.Client: Setting up container launch context for our AM
2020-11-29 15:56:39,948 INFO yarn.Client: Setting up the launch environment for our AM container
2020-11-29 15:56:39,951 INFO yarn.Client: Preparing resources for our AM container
2020-11-29 15:56:39,978 WARN yarn.Client: Neither spark.yarn.jars nor spark.yarn.archive is set, falling back to uploading libraries under SPARK_HOME.
2020-11-29 15:56:40,534 INFO yarn.Client: Uploading resource file:/tmp/spark-49c12823-b6a4-4c2e-b397-d77a78188b8d/__spark_libs__1519230271046889967.zip -> hdfs://localhost:9000/user/mijazz/.sparkStaging/application_1606634326109_0003/__spark_libs__1519230271046889967.zip
2020-11-29 15:56:41,208 INFO yarn.Client: Uploading resource file:/tmp/spark-49c12823-b6a4-4c2e-b397-d77a78188b8d/__spark_conf__3120810522893336741.zip -> hdfs://localhost:9000/user/mijazz/.sparkStaging/application_1606634326109_0003/__spark_conf__.zip
2020-11-29 15:56:41,266 INFO spark.SecurityManager: Changing view acls to: mijazz
2020-11-29 15:56:41,266 INFO spark.SecurityManager: Changing modify acls to: mijazz
2020-11-29 15:56:41,266 INFO spark.SecurityManager: Changing view acls groups to: 
2020-11-29 15:56:41,266 INFO spark.SecurityManager: Changing modify acls groups to: 
2020-11-29 15:56:41,266 INFO spark.SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(mijazz); groups with view permissions: Set(); users  with modify permissions: Set(mijazz); groups with modify permissions: Set()
2020-11-29 15:56:42,004 INFO yarn.Client: Submitting application application_1606634326109_0003 to ResourceManager
2020-11-29 15:56:42,039 INFO impl.YarnClientImpl: Submitted application application_1606634326109_0003
2020-11-29 15:56:42,041 INFO cluster.SchedulerExtensionServices: Starting Yarn extension services with app application_1606634326109_0003 and attemptId None
2020-11-29 15:56:43,046 INFO yarn.Client: Application report for application_1606634326109_0003 (state: ACCEPTED)
2020-11-29 15:56:43,048 INFO yarn.Client: 
         client token: N/A
         diagnostics: AM container is launched, waiting for AM container to Register with RM
         ApplicationMaster host: N/A
         ApplicationMaster RPC port: -1
         queue: default
         start time: 1606636602015
         final status: UNDEFINED
         tracking URL: http://localhost:8088/proxy/application_1606634326109_0003/
         user: mijazz
2020-11-29 15:56:44,050 INFO yarn.Client: Application report for application_1606634326109_0003 (state: ACCEPTED)
2020-11-29 15:56:45,052 INFO yarn.Client: Application report for application_1606634326109_0003 (state: ACCEPTED)
2020-11-29 15:56:45,963 INFO cluster.YarnClientSchedulerBackend: Add WebUI Filter. org.apache.hadoop.yarn.server.webproxy.amfilter.AmIpFilter, Map(PROXY_HOSTS -> localhost, PROXY_URI_BASES -> http://localhost:8088/proxy/application_1606634326109_0003), /proxy/application_1606634326109_0003
2020-11-29 15:56:46,054 INFO yarn.Client: Application report for application_1606634326109_0003 (state: RUNNING)
2020-11-29 15:56:46,054 INFO yarn.Client: 
         client token: N/A
         diagnostics: N/A
         ApplicationMaster host: 192.168.123.2
         ApplicationMaster RPC port: -1
         queue: default
         start time: 1606636602015
         final status: UNDEFINED
         tracking URL: http://localhost:8088/proxy/application_1606634326109_0003/
         user: mijazz
2020-11-29 15:56:46,055 INFO cluster.YarnClientSchedulerBackend: Application application_1606634326109_0003 has started running.
2020-11-29 15:56:46,061 INFO util.Utils: Successfully started service 'org.apache.spark.network.netty.NettyBlockTransferService' on port 37581.
2020-11-29 15:56:46,061 INFO netty.NettyBlockTransferService: Server created on lenovo.lan:37581
2020-11-29 15:56:46,062 INFO storage.BlockManager: Using org.apache.spark.storage.RandomBlockReplicationPolicy for block replication policy
2020-11-29 15:56:46,079 INFO storage.BlockManagerMaster: Registering BlockManager BlockManagerId(driver, lenovo.lan, 37581, None)
2020-11-29 15:56:46,080 INFO storage.BlockManagerMasterEndpoint: Registering block manager lenovo.lan:37581 with 366.3 MB RAM, BlockManagerId(driver, lenovo.lan, 37581, None)
2020-11-29 15:56:46,082 INFO storage.BlockManagerMaster: Registered BlockManager BlockManagerId(driver, lenovo.lan, 37581, None)
2020-11-29 15:56:46,083 INFO storage.BlockManager: Initialized BlockManager: BlockManagerId(driver, lenovo.lan, 37581, None)
2020-11-29 15:56:46,143 INFO cluster.YarnSchedulerBackend$YarnSchedulerEndpoint: ApplicationMaster registered as NettyRpcEndpointRef(spark-client://YarnAM)
2020-11-29 15:56:46,205 INFO ui.JettyUtils: Adding filter org.apache.hadoop.yarn.server.webproxy.amfilter.AmIpFilter to /metrics/json.
2020-11-29 15:56:46,210 INFO handler.ContextHandler: Started o.s.j.s.ServletContextHandler@9e02f84{/metrics/json,null,AVAILABLE,@Spark}
2020-11-29 15:56:46,306 INFO scheduler.EventLoggingListener: Logging events to hdfs://localhost:9000/tmp/spark-logs/application_1606634326109_0003
2020-11-29 15:56:49,276 INFO cluster.YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(spark-client://Executor) (192.168.123.2:37340) with ID 1
2020-11-29 15:56:49,394 INFO storage.BlockManagerMasterEndpoint: Registering block manager localhost:35819 with 366.3 MB RAM, BlockManagerId(1, localhost, 35819, None)
2020-11-29 15:56:49,976 INFO cluster.YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(spark-client://Executor) (192.168.123.2:37344) with ID 2
2020-11-29 15:56:50,034 INFO cluster.YarnClientSchedulerBackend: SchedulerBackend is ready for scheduling beginning after reached minRegisteredResourcesRatio: 0.8
2020-11-29 15:56:50,165 INFO storage.BlockManagerMasterEndpoint: Registering block manager localhost:40629 with 366.3 MB RAM, BlockManagerId(2, localhost, 40629, None)
2020-11-29 15:56:50,304 INFO spark.SparkContext: Starting job: reduce at SparkPi.scala:38
2020-11-29 15:56:50,319 INFO scheduler.DAGScheduler: Got job 0 (reduce at SparkPi.scala:38) with 10 output partitions
2020-11-29 15:56:50,320 INFO scheduler.DAGScheduler: Final stage: ResultStage 0 (reduce at SparkPi.scala:38)
2020-11-29 15:56:50,321 INFO scheduler.DAGScheduler: Parents of final stage: List()
2020-11-29 15:56:50,321 INFO scheduler.DAGScheduler: Missing parents: List()
2020-11-29 15:56:50,325 INFO scheduler.DAGScheduler: Submitting ResultStage 0 (MapPartitionsRDD[1] at map at SparkPi.scala:34), which has no missing parents
2020-11-29 15:56:50,436 INFO memory.MemoryStore: Block broadcast_0 stored as values in memory (estimated size 2.0 KB, free 366.3 MB)
2020-11-29 15:56:50,454 INFO memory.MemoryStore: Block broadcast_0_piece0 stored as bytes in memory (estimated size 1381.0 B, free 366.3 MB)
2020-11-29 15:56:50,456 INFO storage.BlockManagerInfo: Added broadcast_0_piece0 in memory on lenovo.lan:37581 (size: 1381.0 B, free: 366.3 MB)
2020-11-29 15:56:50,459 INFO spark.SparkContext: Created broadcast 0 from broadcast at DAGScheduler.scala:1184
2020-11-29 15:56:50,471 INFO scheduler.DAGScheduler: Submitting 10 missing tasks from ResultStage 0 (MapPartitionsRDD[1] at map at SparkPi.scala:34) (first 15 tasks are for partitions Vector(0, 1, 2, 3, 4, 5, 6, 7, 8, 9))
2020-11-29 15:56:50,471 INFO cluster.YarnScheduler: Adding task set 0.0 with 10 tasks
2020-11-29 15:56:50,496 INFO scheduler.TaskSetManager: Starting task 0.0 in stage 0.0 (TID 0, localhost, executor 2, partition 0, PROCESS_LOCAL, 7877 bytes)
2020-11-29 15:56:50,500 INFO scheduler.TaskSetManager: Starting task 1.0 in stage 0.0 (TID 1, localhost, executor 1, partition 1, PROCESS_LOCAL, 7877 bytes)
2020-11-29 15:56:50,783 INFO storage.BlockManagerInfo: Added broadcast_0_piece0 in memory on localhost:35819 (size: 1381.0 B, free: 366.3 MB)
2020-11-29 15:56:50,976 INFO storage.BlockManagerInfo: Added broadcast_0_piece0 in memory on localhost:40629 (size: 1381.0 B, free: 366.3 MB)
2020-11-29 15:56:51,003 INFO scheduler.TaskSetManager: Starting task 2.0 in stage 0.0 (TID 2, localhost, executor 1, partition 2, PROCESS_LOCAL, 7877 bytes)
2020-11-29 15:56:51,008 INFO scheduler.TaskSetManager: Finished task 1.0 in stage 0.0 (TID 1) in 509 ms on localhost (executor 1) (1/10)
2020-11-29 15:56:51,039 INFO scheduler.TaskSetManager: Starting task 3.0 in stage 0.0 (TID 3, localhost, executor 1, partition 3, PROCESS_LOCAL, 7877 bytes)
2020-11-29 15:56:51,042 INFO scheduler.TaskSetManager: Finished task 2.0 in stage 0.0 (TID 2) in 40 ms on localhost (executor 1) (2/10)
2020-11-29 15:56:51,075 INFO scheduler.TaskSetManager: Starting task 4.0 in stage 0.0 (TID 4, localhost, executor 1, partition 4, PROCESS_LOCAL, 7877 bytes)
2020-11-29 15:56:51,078 INFO scheduler.TaskSetManager: Finished task 3.0 in stage 0.0 (TID 3) in 39 ms on localhost (executor 1) (3/10)
2020-11-29 15:56:51,110 INFO scheduler.TaskSetManager: Starting task 5.0 in stage 0.0 (TID 5, localhost, executor 2, partition 5, PROCESS_LOCAL, 7877 bytes)
2020-11-29 15:56:51,112 INFO scheduler.TaskSetManager: Finished task 0.0 in stage 0.0 (TID 0) in 627 ms on localhost (executor 2) (4/10)
2020-11-29 15:56:51,122 INFO scheduler.TaskSetManager: Starting task 6.0 in stage 0.0 (TID 6, localhost, executor 1, partition 6, PROCESS_LOCAL, 7877 bytes)
2020-11-29 15:56:51,123 INFO scheduler.TaskSetManager: Finished task 4.0 in stage 0.0 (TID 4) in 47 ms on localhost (executor 1) (5/10)
2020-11-29 15:56:51,166 INFO scheduler.TaskSetManager: Starting task 7.0 in stage 0.0 (TID 7, localhost, executor 1, partition 7, PROCESS_LOCAL, 7877 bytes)
2020-11-29 15:56:51,171 INFO scheduler.TaskSetManager: Finished task 6.0 in stage 0.0 (TID 6) in 49 ms on localhost (executor 1) (6/10)
2020-11-29 15:56:51,171 INFO scheduler.TaskSetManager: Starting task 8.0 in stage 0.0 (TID 8, localhost, executor 2, partition 8, PROCESS_LOCAL, 7877 bytes)
2020-11-29 15:56:51,172 INFO scheduler.TaskSetManager: Finished task 5.0 in stage 0.0 (TID 5) in 62 ms on localhost (executor 2) (7/10)
2020-11-29 15:56:51,187 INFO scheduler.TaskSetManager: Starting task 9.0 in stage 0.0 (TID 9, localhost, executor 1, partition 9, PROCESS_LOCAL, 7877 bytes)
2020-11-29 15:56:51,188 INFO scheduler.TaskSetManager: Finished task 7.0 in stage 0.0 (TID 7) in 22 ms on localhost (executor 1) (8/10)
2020-11-29 15:56:51,213 INFO scheduler.TaskSetManager: Finished task 8.0 in stage 0.0 (TID 8) in 42 ms on localhost (executor 2) (9/10)
2020-11-29 15:56:51,219 INFO scheduler.TaskSetManager: Finished task 9.0 in stage 0.0 (TID 9) in 33 ms on localhost (executor 1) (10/10)
2020-11-29 15:56:51,220 INFO cluster.YarnScheduler: Removed TaskSet 0.0, whose tasks have all completed, from pool 
2020-11-29 15:56:51,221 INFO scheduler.DAGScheduler: ResultStage 0 (reduce at SparkPi.scala:38) finished in 0.868 s
2020-11-29 15:56:51,224 INFO scheduler.DAGScheduler: Job 0 finished: reduce at SparkPi.scala:38, took 0.920297 s
Pi is roughly 3.1402631402631402
2020-11-29 15:56:51,235 INFO server.AbstractConnector: Stopped Spark@33aa93c{HTTP/1.1,[http/1.1]}{0.0.0.0:4040}
2020-11-29 15:56:51,237 INFO ui.SparkUI: Stopped Spark web UI at http://lenovo.lan:4040
2020-11-29 15:56:51,240 INFO cluster.YarnClientSchedulerBackend: Interrupting monitor thread
2020-11-29 15:56:51,263 INFO cluster.YarnClientSchedulerBackend: Shutting down all executors
2020-11-29 15:56:51,264 INFO cluster.YarnSchedulerBackend$YarnDriverEndpoint: Asking each executor to shut down
2020-11-29 15:56:51,268 INFO cluster.SchedulerExtensionServices: Stopping SchedulerExtensionServices
(serviceOption=None,
 services=List(),
 started=false)
2020-11-29 15:56:51,268 INFO cluster.YarnClientSchedulerBackend: Stopped
2020-11-29 15:56:51,364 INFO spark.MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
2020-11-29 15:56:51,371 INFO memory.MemoryStore: MemoryStore cleared
2020-11-29 15:56:51,371 INFO storage.BlockManager: BlockManager stopped
2020-11-29 15:56:51,374 INFO storage.BlockManagerMaster: BlockManagerMaster stopped
2020-11-29 15:56:51,376 INFO scheduler.OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
2020-11-29 15:56:51,398 INFO spark.SparkContext: Successfully stopped SparkContext
2020-11-29 15:56:51,400 INFO util.ShutdownHookManager: Shutdown hook called
2020-11-29 15:56:51,401 INFO util.ShutdownHookManager: Deleting directory /tmp/spark-5c235e44-e3d3-4d12-923c-a635b9143c39
2020-11-29 15:56:51,403 INFO util.ShutdownHookManager: Deleting directory /tmp/spark-49c12823-b6a4-4c2e-b397-d77a78188b8d
```

