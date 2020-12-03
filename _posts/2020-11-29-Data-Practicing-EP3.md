---
title: Data Practicing-EP3
author: MijazzChan
date: 2020-11-29 20:10:31 +0800
categories: [Data, Spark]
tags: [data, spark, java]

---

# Data Practicing-EP3

## Introduce Spark

> 这里贴出几个官方文档
>
> [Spark Overview](https://spark.apache.org/docs/2.4.7/)
>
> [Java API Docs](https://spark.apache.org/docs/2.4.7/api/java/index.html)
>
> [Scala API Docs](https://spark.apache.org/docs/2.4.7/api/scala/index.html)
>
> [Spark SQL Docs](https://spark.apache.org/docs/2.4.7/api/sql/index.html)

这里只记录一下`SparkRDD`, `RDD` -> `Resilient Distributed Datasets`.

它是一种可扩展的弹性分布式数据集, 他是只读的, 分区的, 并且保持不变的数据集合, 直接与在内存层面的一个分布式实现.

+ 可分区/片(~~默认好象是Hash分区?~~)
+ 可自定义分片计算函数
+ 互相依赖(下个分区由之前的分区通过转换生成)
+ 可控制分片数量
+ 可以使用列表方式进行块储存

它支持两种类型的操作

+ Transformations
  - `map()`
  - `flatMap()`
  - `filter()`
  - `union()`
  - `intersection()`
  - ......
+ Actions
  - `reduce()`
  - `collect()`
  - `count()`
  - ......

### `RDD` Operations Examples

> 以下Code Block均为在`Spark-shell`下执行的结果
>
> `Spark`-> 2.4.7 `Yarn` on `Hadoop 3.1.4`
>
> `Scala version 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_275)`

```scala
scala> val data = Array(2, 3, 5, 7, 11)
data: Array[Int] = Array(2, 3, 5, 7, 11)

scala> val rdd1 = sc.parallelize(data)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:26

scala> val rdd2 = rdd1.map(element => (element*2, element*element)).collect()
rdd2: Array[(Int, Int)] = Array((4,4), (6,9), (10,25), (14,49), (22,121))

scala> val rdd3 = rdd1.union(rdd1)
rdd3: org.apache.spark.rdd.RDD[Int] = UnionRDD[3] at union at <console>:25

scala> rdd3.collect()
res4: Array[Int] = Array(2, 3, 5, 7, 11, 2, 3, 5, 7, 11)

scala> rdd3.sortBy(x => x%8, ascending=false).collect()
res5: Array[Int] = Array(7, 7, 5, 5, 11, 3, 3, 11, 2, 2)

scala> rdd3.count()
res6: Long = 10

scala> rdd3.take(3)
res7: Array[Int] = Array(2, 3, 5)

scala> rdd3.distinct().collect()
res8: Array[Int] = Array(5, 2, 11, 3, 7)
........
```

加上之前我们在`hadoop`里运行的`HDFS`, Spark可以很方便的通过`hdfs://ip.or.host:port/path/to/file`来访问`hdfs`的文件.

也可以使用`spark sql`在处理数据.



## Prepare to Code

网上太多教材关于`Spark` + `Scala` + `IntelliJ IDEA` + `sbt`四大件的了

贴几个教程链接

> [IntelliJ IDEA sbt](https://www.jetbrains.com/help/idea/sbt-support.html#sbt_structure)
>
> [IntelliJ IDEA Scala](https://www.jetbrains.com/help/idea/discover-intellij-idea-for-scala.html)
>
> [Scala Official - Dev with IDEA](https://docs.scala-lang.org/getting-started/intellij-track/getting-started-with-scala-in-intellij.html)
>
> [Tutorial 1](https://learnscalaspark.com/getting-started-intellij-scala-apache-spark)
>
> [Tutorial-2](https://medium.com/@mrpowers/creating-a-spark-project-with-sbt-intellij-sbt-spark-package-and-friends-cc9108751c28)

包管理对于我来说, 还是更熟悉`Java`的那一套, 毕竟`Spring`用多了. 不是`Maven`就是`Gradle`.

镜像设置过程不表, 见[Aliyun Maven Mirror](https://developer.aliyun.com/mirror/maven)

> Manjaro Linux 20
>
> IntelliJ IDEA Ultimate 2020.2.3
>
> Maven(bundled with idea, 3.6.3)
>
> Install Scala Plugin in IDEA(!important)

常规IDEA建立`Maven`的`Project`, 依赖如下

`pom.xml`

```xml
    <dependencies>
    <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-core -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>2.4.7</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-sql -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.11</artifactId>
            <version>2.4.7</version>
        </dependency>

    </dependencies>

```

如果spark版本不同, 去`mvnrepository`搜索对应的依赖, 粘贴进依赖区即可.

Sync一下, Maven即可解决依赖问题. 而后在工程下右键, `Add Framework Support`, 加入该工程对`Scala`的支持.(该步骤需要有`Scala Plugin`)

常规建包建类即可, 选`Scala Class -> Object`

> 参考[Naming Conventions](https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html)

我的步骤

+ $Project Root/src/main/java新建`package` -> `edu.zstu.mijazz.sparklearn`

+ 包下建类`Scala Object -> HelloWorld`

+ ```scala
  package edu.zstu.mijazz.sparklearn1
  
  import org.apache.spark.sql.SparkSession
  
  import scala.math.random
  
  object HelloWorld {
    def main(args: Array[String]): Unit = {
      val spark = SparkSession.builder.appName("Spark Pi").master("local").getOrCreate()
      val count = spark.sparkContext.parallelize(1 until 50000000, 3).map {_ =>
        val x = random * 2 - 1
        val y = random * 2 - 1
        if (x*x + y*y <= 1) 1 else 0
      }.reduce(_ + _)
      println(s"Pi is roughly ${4.0 * count / (50000000 - 1)}")
      spark.stop()
      spark.close()
    }
  }
  ```

  直接建object, 执行时对象初始化触发对象`main()`, 至于Scala的语法和资料, 见[Scala Docs](https://docs.scala-lang.org/overviews/)

+ 如果输出没问题

  ```scala
  Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
  20/11/29 22:21:24 INFO SparkContext: Running Spark version 2.4.7
  20/11/29 22:21:24 INFO SparkContext: Submitted application: Spark Pi
  20/11/29 22:21:24 INFO SecurityManager: Changing view acls to: mijazz
  20/11/29 22:21:24 INFO SecurityManager: Changing modify acls to: mijazz
  20/11/29 22:21:24 INFO SecurityManager: Changing view acls groups to: 
  20/11/29 22:21:24 INFO SecurityManager: Changing modify acls groups to: 
  20/11/29 22:21:24 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(mijazz); groups with view permissions: Set(); users  with modify permissions: Set(mijazz); groups with modify permissions: Set()
  20/11/29 22:21:24 INFO Utils: Successfully started service 'sparkDriver' on port 46007.
  20/11/29 22:21:24 INFO SparkEnv: Registering MapOutputTracker
  20/11/29 22:21:24 INFO SparkEnv: Registering BlockManagerMaster
  20/11/29 22:21:25 INFO BlockManagerMasterEndpoint: Using org.apache.spark.storage.DefaultTopologyMapper for getting topology information
  20/11/29 22:21:25 INFO BlockManagerMasterEndpoint: BlockManagerMasterEndpoint up
  20/11/29 22:21:25 INFO DiskBlockManager: Created local directory at /tmp/blockmgr-be049433-3f00-4037-a865-67cd6f445fba
  20/11/29 22:21:25 INFO MemoryStore: MemoryStore started with capacity 1941.6 MB
  20/11/29 22:21:25 INFO SparkEnv: Registering OutputCommitCoordinator
  20/11/29 22:21:25 INFO Utils: Successfully started service 'SparkUI' on port 4040.
  20/11/29 22:21:25 INFO SparkUI: Bound SparkUI to 0.0.0.0, and started at http://lenovo.lan:4040
  20/11/29 22:21:25 INFO Executor: Starting executor ID driver on host localhost
  20/11/29 22:21:25 INFO Utils: Successfully started service 'org.apache.spark.network.netty.NettyBlockTransferService' on port 44811.
  20/11/29 22:21:25 INFO NettyBlockTransferService: Server created on lenovo.lan:44811
  20/11/29 22:21:25 INFO BlockManager: Using org.apache.spark.storage.RandomBlockReplicationPolicy for block replication policy
  20/11/29 22:21:25 INFO BlockManagerMaster: Registering BlockManager BlockManagerId(driver, lenovo.lan, 44811, None)
  20/11/29 22:21:25 INFO BlockManagerMasterEndpoint: Registering block manager lenovo.lan:44811 with 1941.6 MB RAM, BlockManagerId(driver, lenovo.lan, 44811, None)
  20/11/29 22:21:25 INFO BlockManagerMaster: Registered BlockManager BlockManagerId(driver, lenovo.lan, 44811, None)
  20/11/29 22:21:25 INFO BlockManager: Initialized BlockManager: BlockManagerId(driver, lenovo.lan, 44811, None)
  20/11/29 22:21:25 INFO SparkContext: Starting job: reduce at HelloWorld.scala:15
  20/11/29 22:21:26 INFO DAGScheduler: Got job 0 (reduce at HelloWorld.scala:15) with 3 output partitions
  20/11/29 22:21:26 INFO DAGScheduler: Final stage: ResultStage 0 (reduce at HelloWorld.scala:15)
  20/11/29 22:21:26 INFO DAGScheduler: Parents of final stage: List()
  20/11/29 22:21:26 INFO DAGScheduler: Missing parents: List()
  20/11/29 22:21:26 INFO DAGScheduler: Submitting ResultStage 0 (MapPartitionsRDD[1] at map at HelloWorld.scala:11), which has no missing parents
  20/11/29 22:21:26 INFO MemoryStore: Block broadcast_0 stored as values in memory (estimated size 2.0 KB, free 1941.6 MB)
  20/11/29 22:21:26 INFO MemoryStore: Block broadcast_0_piece0 stored as bytes in memory (estimated size 1401.0 B, free 1941.6 MB)
  20/11/29 22:21:26 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on lenovo.lan:44811 (size: 1401.0 B, free: 1941.6 MB)
  20/11/29 22:21:26 INFO SparkContext: Created broadcast 0 from broadcast at DAGScheduler.scala:1184
  20/11/29 22:21:26 INFO DAGScheduler: Submitting 3 missing tasks from ResultStage 0 (MapPartitionsRDD[1] at map at HelloWorld.scala:11) (first 15 tasks are for partitions Vector(0, 1, 2))
  20/11/29 22:21:26 INFO TaskSchedulerImpl: Adding task set 0.0 with 3 tasks
  20/11/29 22:21:26 INFO TaskSetManager: Starting task 0.0 in stage 0.0 (TID 0, localhost, executor driver, partition 0, PROCESS_LOCAL, 7866 bytes)
  20/11/29 22:21:26 INFO Executor: Running task 0.0 in stage 0.0 (TID 0)
  20/11/29 22:21:27 INFO Executor: Finished task 0.0 in stage 0.0 (TID 0). 867 bytes result sent to driver
  20/11/29 22:21:27 INFO TaskSetManager: Starting task 1.0 in stage 0.0 (TID 1, localhost, executor driver, partition 1, PROCESS_LOCAL, 7866 bytes)
  20/11/29 22:21:27 INFO Executor: Running task 1.0 in stage 0.0 (TID 1)
  20/11/29 22:21:27 INFO TaskSetManager: Finished task 0.0 in stage 0.0 (TID 0) in 1038 ms on localhost (executor driver) (1/3)
  20/11/29 22:21:28 INFO Executor: Finished task 1.0 in stage 0.0 (TID 1). 867 bytes result sent to driver
  20/11/29 22:21:28 INFO TaskSetManager: Starting task 2.0 in stage 0.0 (TID 2, localhost, executor driver, partition 2, PROCESS_LOCAL, 7866 bytes)
  20/11/29 22:21:28 INFO Executor: Running task 2.0 in stage 0.0 (TID 2)
  20/11/29 22:21:28 INFO TaskSetManager: Finished task 1.0 in stage 0.0 (TID 1) in 998 ms on localhost (executor driver) (2/3)
  20/11/29 22:21:29 INFO Executor: Finished task 2.0 in stage 0.0 (TID 2). 867 bytes result sent to driver
  20/11/29 22:21:29 INFO TaskSetManager: Finished task 2.0 in stage 0.0 (TID 2) in 991 ms on localhost (executor driver) (3/3)
  20/11/29 22:21:29 INFO TaskSchedulerImpl: Removed TaskSet 0.0, whose tasks have all completed, from pool 
  20/11/29 22:21:29 INFO DAGScheduler: ResultStage 0 (reduce at HelloWorld.scala:15) finished in 3.169 s
  20/11/29 22:21:29 INFO DAGScheduler: Job 0 finished: reduce at HelloWorld.scala:15, took 3.204133 s
  20/11/29 22:21:29 INFO SparkUI: Stopped Spark web UI at http://lenovo.lan:4040
  Pi is roughly 3.141491662829833
  20/11/29 22:21:29 INFO MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
  20/11/29 22:21:29 INFO MemoryStore: MemoryStore cleared
  20/11/29 22:21:29 INFO BlockManager: BlockManager stopped
  20/11/29 22:21:29 INFO BlockManagerMaster: BlockManagerMaster stopped
  20/11/29 22:21:29 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
  20/11/29 22:21:29 INFO SparkContext: Successfully stopped SparkContext
  20/11/29 22:21:29 INFO SparkContext: SparkContext already stopped.
  20/11/29 22:21:29 INFO ShutdownHookManager: Shutdown hook called
  20/11/29 22:21:29 INFO ShutdownHookManager: Deleting directory /tmp/spark-701b3922-5c91-4ada-80de-0319be2db7e3
  ```

  

能够跑出结果, 说明在`IDEA`中直接使用`scala`与`spark`交互已经没问题了. 现在开始找数据集试试`DataFrame`

