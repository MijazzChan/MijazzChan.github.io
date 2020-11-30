---
title: Hadoop+Spark Data Practicing-EP4
author: MijazzChan
date: 2020-11-29 22:48:19 +0800
categories: [Data, Hadoop]
tags: [data, hadoop, spark, java]


---

# Hadoop+Spark Data Practicing-EP4

## Find Data

> Chicago Crime Data is from `CHICAGO DATA PORTAL`
>
> [Visit Here](https://data.cityofchicago.org)

这次使用的是Chicago的Crime Data. 从2001年至最近的.

```shell
 mijazz@lenovo  ~/devEnvs  wc -l chicagoCrimeData.csv
7212274 chicagoCrimeData.csv
 mijazz@lenovo  ~/devEnvs  head -n 2 ./chicagoCrimeData.csv 
ID,Case Number,Date,Block,IUCR,Primary Type,Description,Location Description,Arrest,Domestic,Beat,District,Ward,Community Area,FBI Code,X Coordinate,Y Coordinate,Year,Updated On,Latitude,Longitude,Location
11034701,JA366925,01/01/2001 11:00:00 AM,016XX E 86TH PL,1153,DECEPTIVE PRACTICE,FINANCIAL IDENTITY THEFT OVER $ 300,RESIDENCE,false,false,0412,004,8,45,11,,,2001,08/05/2017 03:50:08 PM,,,
 mijazz@lenovo  ~/devEnvs  tail -n 2 ./chicagoCrimeData.csv 
11707239,JC287563,11/30/2017 09:00:00 AM,022XX S KOSTNER AVE,1153,DECEPTIVE PRACTICE,FINANCIAL IDENTITY THEFT OVER $ 300,RESIDENCE,false,false,1013,010,22,29,11,,,2017,06/02/2019 04:09:42 PM,,,
24559,JC278908,05/26/2019 02:11:00 AM,013XX W HASTINGS ST,0110,HOMICIDE,FIRST DEGREE MURDER,STREET,false,false,1233,012,25,28,01A,1167746,1893853,2019,06/20/2020 03:48:45 PM,41.864278357,-87.659682244,"(41.864278357, -87.659682244)"
```

共`7,212,274`行数据, 每行数据代表一次记录在案的犯罪信息. 

部分列描述如下

+ ID - Unique Row ID
+ Case Number - Unique Chicago Police Department Records Division Number, Unique
+ Date
+ Block - Address
+ IUCR - Illinois Uniform Crime Reporting Code[Code Referrence](https://data.cityofchicago.org/Public-Safety/Chicago-Police-Department-Illinois-Uniform-Crime-R/c7ck-438e)
+ Primary Type - IUCR Code/Crime Description
+ Description - Crime Description
+ Location Description
+ Arrest - Arrest made or not
+ Community Area - Community Area Code [Code Referrence](https://data.cityofchicago.org/Facilities-Geographic-Boundaries/Boundaries-Community-Areas-current-/cauq-8yn6)
+ Location - (Latitude, Longitude)



## Move to HDFS

> 前面说过`hdfs`的提供的交互`shell`很像`Unix/Linux`的文件系统交互.
>
> 文档如下: [File System Shell](https://hadoop.apache.org/docs/r3.1.4/hadoop-project-dist/hadoop-common/FileSystemShell.html) 或者 `hdfs dfs -help`

```shell
 mijazz@lenovo  ~/devEnvs  ll -a
total 1.8G
drwxr-xr-x 10 mijazz mijazz 4.0K Nov 29 22:45 .
drwx------ 42 mijazz mijazz 4.0K Nov 30 15:42 ..
drwxr-xr-x  8 mijazz mijazz 4.0K Nov  9 20:23 OpenJDK8
-rwxrwxrwx  1 mijazz mijazz 1.6G Oct 19 18:05 chicagoCrimeData.csv
drwxr-xr-x 11 mijazz mijazz 4.0K Nov 25 23:49 hadoop-3.1.4
drwxr-xr-x 14 mijazz mijazz 4.0K Nov 29 15:25 spark-2.4.7
-rw-r--r--  1 mijazz mijazz 161M Nov 24 16:49 spark-2.4.7-bin-without-hadoop.tgz
 mijazz@lenovo  ~/devEnvs  hdfs dfs -mkdir /user/mijazz/chicagoData                
 mijazz@lenovo  ~/devEnvs  hdfs dfs -put ./chicagoCrimeData.csv /user/mijazz/chicagoData/originCrimeData.csv 
 mijazz@lenovo  ~/devEnvs  hdfs dfs -ls /user/mijazz/chicagoData                  
Found 1 items
-rw-r--r--   1 mijazz supergroup 1701238602 2020-11-30 15:43 /user/mijazz/chicagoData/originCrimeData.csv

```

`dfs -put`把文件上传上hdfs, 如果需要~~多用户读写, `dfs -chmod`给个`666`之后,~~ 检查一下权限即可.

上传之后, 在spark中就可以通过`hdfs://your.ip.or.host:port/path/to/file`来访问了.

在我这里就是`hdfs://localhost:9000/user/mijazz/chicagoData/originCrimeData.csv`



## Pre-Processing

在EP3中配置好的`IntelliJ IDEA`的project, 新建一个`Scala Object`即可.

前面几行代码都是必备的了

+ 用SparkSession拉起一个Spark会话
+ Context负责数据

### Take a Glance at the Data

> Object + main()方法或者Object + extends App 当脚本用

`DataPreProcess.scala` - 1

```scala
package edu.zstu.mijazz.sparklearn1

import org.apache.spark.sql.SparkSession

object DataPreProcess {
  val HDFS_PATH = "hdfs://localhost:9000/user/mijazz/"
  val DATA_PATH = HDFS_PATH + "chicagoData/"
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder.appName("Data Pre-Processing").master("local").getOrCreate()
    val sContext = spark.sparkContext

    val data = sContext.textFile(DATA_PATH + "originCrimeData.csv")
    data.take(3).foreach(println)
  }
}

```

#### Full Output

> 后面的输出block, 我会把spark的输出手工砍掉. ~~当然你也可以在spark中配置比`info`级别高一些的log level, 但是留着便于我知道内存使用量.~~

```java
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
20/11/30 16:00:51 WARN Utils: Your hostname, lenovo resolves to a loopback address: 127.0.0.1; using 192.168.123.2 instead (on interface enp4s0)
20/11/30 16:00:51 WARN Utils: Set SPARK_LOCAL_IP if you need to bind to another address
20/11/30 16:00:51 INFO SparkContext: Running Spark version 2.4.7
20/11/30 16:00:51 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
20/11/30 16:00:51 INFO SparkContext: Submitted application: Data Pre-Processing
20/11/30 16:00:51 INFO SecurityManager: Changing view acls to: mijazz
20/11/30 16:00:51 INFO SecurityManager: Changing modify acls to: mijazz
20/11/30 16:00:51 INFO SecurityManager: Changing view acls groups to: 
20/11/30 16:00:51 INFO SecurityManager: Changing modify acls groups to: 
20/11/30 16:00:51 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(mijazz); groups with view permissions: Set(); users  with modify permissions: Set(mijazz); groups with modify permissions: Set()
20/11/30 16:00:52 INFO Utils: Successfully started service 'sparkDriver' on port 35377.
20/11/30 16:00:52 INFO SparkEnv: Registering MapOutputTracker
20/11/30 16:00:52 INFO SparkEnv: Registering BlockManagerMaster
20/11/30 16:00:52 INFO BlockManagerMasterEndpoint: Using org.apache.spark.storage.DefaultTopologyMapper for getting topology information
20/11/30 16:00:52 INFO BlockManagerMasterEndpoint: BlockManagerMasterEndpoint up
20/11/30 16:00:52 INFO DiskBlockManager: Created local directory at /tmp/blockmgr-283f5487-ed7e-41b8-92ae-20d56fb33ba5
20/11/30 16:00:52 INFO MemoryStore: MemoryStore started with capacity 1941.6 MB
20/11/30 16:00:52 INFO SparkEnv: Registering OutputCommitCoordinator
20/11/30 16:00:52 INFO Utils: Successfully started service 'SparkUI' on port 4040.
20/11/30 16:00:52 INFO SparkUI: Bound SparkUI to 0.0.0.0, and started at http://lenovo.lan:4040
20/11/30 16:00:52 INFO Executor: Starting executor ID driver on host localhost
20/11/30 16:00:52 INFO Utils: Successfully started service 'org.apache.spark.network.netty.NettyBlockTransferService' on port 37555.
20/11/30 16:00:52 INFO NettyBlockTransferService: Server created on lenovo.lan:37555
20/11/30 16:00:52 INFO BlockManager: Using org.apache.spark.storage.RandomBlockReplicationPolicy for block replication policy
20/11/30 16:00:52 INFO BlockManagerMaster: Registering BlockManager BlockManagerId(driver, lenovo.lan, 37555, None)
20/11/30 16:00:52 INFO BlockManagerMasterEndpoint: Registering block manager lenovo.lan:37555 with 1941.6 MB RAM, BlockManagerId(driver, lenovo.lan, 37555, None)
20/11/30 16:00:52 INFO BlockManagerMaster: Registered BlockManager BlockManagerId(driver, lenovo.lan, 37555, None)
20/11/30 16:00:52 INFO BlockManager: Initialized BlockManager: BlockManagerId(driver, lenovo.lan, 37555, None)
20/11/30 16:00:53 INFO MemoryStore: Block broadcast_0 stored as values in memory (estimated size 214.6 KB, free 1941.4 MB)
20/11/30 16:00:53 INFO MemoryStore: Block broadcast_0_piece0 stored as bytes in memory (estimated size 20.4 KB, free 1941.4 MB)
20/11/30 16:00:53 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on lenovo.lan:37555 (size: 20.4 KB, free: 1941.6 MB)
20/11/30 16:00:53 INFO SparkContext: Created broadcast 0 from textFile at DataPreProcess.scala:12
20/11/30 16:00:53 INFO FileInputFormat: Total input paths to process : 1
20/11/30 16:00:53 INFO SparkContext: Starting job: take at DataPreProcess.scala:13
20/11/30 16:00:53 INFO DAGScheduler: Got job 0 (take at DataPreProcess.scala:13) with 1 output partitions
20/11/30 16:00:53 INFO DAGScheduler: Final stage: ResultStage 0 (take at DataPreProcess.scala:13)
20/11/30 16:00:53 INFO DAGScheduler: Parents of final stage: List()
20/11/30 16:00:53 INFO DAGScheduler: Missing parents: List()
20/11/30 16:00:53 INFO DAGScheduler: Submitting ResultStage 0 (hdfs://localhost:9000/user/mijazz/chicagoData/originCrimeData.csv MapPartitionsRDD[1] at textFile at DataPreProcess.scala:12), which has no missing parents
20/11/30 16:00:53 INFO MemoryStore: Block broadcast_1 stored as values in memory (estimated size 3.6 KB, free 1941.4 MB)
20/11/30 16:00:53 INFO MemoryStore: Block broadcast_1_piece0 stored as bytes in memory (estimated size 2.2 KB, free 1941.4 MB)
20/11/30 16:00:53 INFO BlockManagerInfo: Added broadcast_1_piece0 in memory on lenovo.lan:37555 (size: 2.2 KB, free: 1941.6 MB)
20/11/30 16:00:53 INFO SparkContext: Created broadcast 1 from broadcast at DAGScheduler.scala:1184
20/11/30 16:00:53 INFO DAGScheduler: Submitting 1 missing tasks from ResultStage 0 (hdfs://localhost:9000/user/mijazz/chicagoData/originCrimeData.csv MapPartitionsRDD[1] at textFile at DataPreProcess.scala:12) (first 15 tasks are for partitions Vector(0))
20/11/30 16:00:53 INFO TaskSchedulerImpl: Adding task set 0.0 with 1 tasks
20/11/30 16:00:54 INFO TaskSetManager: Starting task 0.0 in stage 0.0 (TID 0, localhost, executor driver, partition 0, PROCESS_LOCAL, 7925 bytes)
20/11/30 16:00:54 INFO Executor: Running task 0.0 in stage 0.0 (TID 0)
20/11/30 16:00:54 INFO HadoopRDD: Input split: hdfs://localhost:9000/user/mijazz/chicagoData/originCrimeData.csv:0+134217728
20/11/30 16:00:54 INFO Executor: Finished task 0.0 in stage 0.0 (TID 0). 1371 bytes result sent to driver
20/11/30 16:00:54 INFO TaskSetManager: Finished task 0.0 in stage 0.0 (TID 0) in 133 ms on localhost (executor driver) (1/1)
20/11/30 16:00:54 INFO TaskSchedulerImpl: Removed TaskSet 0.0, whose tasks have all completed, from pool 
20/11/30 16:00:54 INFO DAGScheduler: ResultStage 0 (take at DataPreProcess.scala:13) finished in 0.183 s
20/11/30 16:00:54 INFO DAGScheduler: Job 0 finished: take at DataPreProcess.scala:13, took 0.217644 s
20/11/30 16:00:54 INFO SparkContext: Invoking stop() from shutdown hook
ID,Case Number,Date,Block,IUCR,Primary Type,Description,Location Description,Arrest,Domestic,Beat,District,Ward,Community Area,FBI Code,X Coordinate,Y Coordinate,Year,Updated On,Latitude,Longitude,Location
11034701,JA366925,01/01/2001 11:00:00 AM,016XX E 86TH PL,1153,DECEPTIVE PRACTICE,FINANCIAL IDENTITY THEFT OVER $ 300,RESIDENCE,false,false,0412,004,8,45,11,,,2001,08/05/2017 03:50:08 PM,,,
11227287,JB147188,10/08/2017 03:00:00 AM,092XX S RACINE AVE,0281,CRIM SEXUAL ASSAULT,NON-AGGRAVATED,RESIDENCE,false,false,2222,022,21,73,02,,,2017,02/11/2018 03:57:41 PM,,,
20/11/30 16:00:54 INFO SparkUI: Stopped Spark web UI at http://lenovo.lan:4040
20/11/30 16:00:54 INFO MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
20/11/30 16:00:54 INFO MemoryStore: MemoryStore cleared
20/11/30 16:00:54 INFO BlockManager: BlockManager stopped
20/11/30 16:00:54 INFO BlockManagerMaster: BlockManagerMaster stopped
20/11/30 16:00:54 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
20/11/30 16:00:54 INFO SparkContext: Successfully stopped SparkContext
20/11/30 16:00:54 INFO ShutdownHookManager: Shutdown hook called
20/11/30 16:00:54 INFO ShutdownHookManager: Deleting directory /tmp/spark-c6ba3bd3-3fc3-46b8-9e7e-3413981456ff

Process finished with exit code 0

```

#### Useful Output

```java
ID,Case Number,Date,Block,IUCR,Primary Type,Description,Location Description,Arrest,Domestic,Beat,District,Ward,Community Area,FBI Code,X Coordinate,Y Coordinate,Year,Updated On,Latitude,Longitude,Location
11034701,JA366925,01/01/2001 11:00:00 AM,016XX E 86TH PL,1153,DECEPTIVE PRACTICE,FINANCIAL IDENTITY THEFT OVER $ 300,RESIDENCE,false,false,0412,004,8,45,11,,,2001,08/05/2017 03:50:08 PM,,,
11227287,JB147188,10/08/2017 03:00:00 AM,092XX S RACINE AVE,0281,CRIM SEXUAL ASSAULT,NON-AGGRAVATED,RESIDENCE,false,false,2222,022,21,73,02,,,2017,02/11/2018 03:57:41 PM,,,
```

有不少空值.



`DataPreProcess.scala` - 2

```scala
package edu.zstu.mijazz.sparklearn1

import org.apache.spark.sql.SparkSession

object DataPreProcess extends App {
  val HDFS_PATH = "hdfs://localhost:9000/user/mijazz/"
  val DATA_PATH = HDFS_PATH + "chicagoData/"
  val spark = SparkSession.builder.appName("Data Pre-Processing").master("local").getOrCreate()
  val sContext = spark.sparkContext

  val crimeDataFrame = spark.read
    .option("header", true)
    .option("inferSchema", true)
    .csv(DATA_PATH + "originCrimeData.csv")

  crimeDataFrame.show(3)
  crimeDataFrame.printSchema()

  spark.stop()
  spark.close()
}
```

```
+--------+-----------+--------------------+------------------+----+-------------------+--------------------+--------------------+------+--------+----+--------+----+--------------+--------+------------+------------+----+--------------------+--------+---------+--------+
|      ID|Case Number|                Date|             Block|IUCR|       Primary Type|         Description|Location Description|Arrest|Domestic|Beat|District|Ward|Community Area|FBI Code|X Coordinate|Y Coordinate|Year|          Updated On|Latitude|Longitude|Location|
+--------+-----------+--------------------+------------------+----+-------------------+--------------------+--------------------+------+--------+----+--------+----+--------------+--------+------------+------------+----+--------------------+--------+---------+--------+
|11034701|   JA366925|01/01/2001 11:00:...|   016XX E 86TH PL|1153| DECEPTIVE PRACTICE|FINANCIAL IDENTIT...|           RESIDENCE| false|   false| 412|       4|   8|            45|      11|        null|        null|2001|08/05/2017 03:50:...|    null|     null|    null|
|11227287|   JB147188|10/08/2017 03:00:...|092XX S RACINE AVE|0281|CRIM SEXUAL ASSAULT|      NON-AGGRAVATED|           RESIDENCE| false|   false|2222|      22|  21|            73|      02|        null|        null|2017|02/11/2018 03:57:...|    null|     null|    null|
|11227583|   JB147595|03/28/2017 02:00:...|   026XX W 79TH ST|0620|           BURGLARY|      UNLAWFUL ENTRY|               OTHER| false|   false| 835|       8|  18|            70|      05|        null|        null|2017|02/11/2018 03:57:...|    null|     null|    null|
+--------+-----------+--------------------+------------------+----+-------------------+--------------------+--------------------+------+--------+----+--------+----+--------------+--------+------------+------------+----+--------------------+--------+---------+--------+
only showing top 3 rows

root
 |-- ID: integer (nullable = true)
 |-- Case Number: string (nullable = true)
 |-- Date: string (nullable = true)
 |-- Block: string (nullable = true)
 |-- IUCR: string (nullable = true)
 |-- Primary Type: string (nullable = true)
 |-- Description: string (nullable = true)
 |-- Location Description: string (nullable = true)
 |-- Arrest: boolean (nullable = true)
 |-- Domestic: boolean (nullable = true)
 |-- Beat: integer (nullable = true)
 |-- District: integer (nullable = true)
 |-- Ward: integer (nullable = true)
 |-- Community Area: integer (nullable = true)
 |-- FBI Code: string (nullable = true)
 |-- X Coordinate: integer (nullable = true)
 |-- Y Coordinate: integer (nullable = true)
 |-- Year: integer (nullable = true)
 |-- Updated On: string (nullable = true)
 |-- Latitude: double (nullable = true)
 |-- Longitude: double (nullable = true)
 |-- Location: string (nullable = true)

```

## Start with Data

为了方便阅读, 多加了一个`Java Class` - `StaticTool.java`, 专门用来存静态数据.

```java
package edu.zstu.mijazz.sparklearn1;

public class StaticTool {
    public static final String HDFS_PATH = "hdfs://localhost:9000/user/mijazz/";
    public static final String DATA_PATH = HDFS_PATH + "chicagoData/";
    public static final String ORIGIN_DATA = DATA_PATH + "originCrimeData.csv";
    public static final String DATE_DATA = DATA_PATH + "dateDF.csv";
}
```

### Date Column

万事就先从时间开始吧, 对`Date`字段先做个分析

```scala
  val dateNullRowCount = crimeDataFrame.filter("Date is null").count()
  println(dateNullRowCount)
```

```scala
0
```

看到日期值并没有空列, 很好, 不用`na.fill`了.

`foucsDate.scala` - 1

```scala
package edu.zstu.mijazz.sparklearn1

import org.apache.spark.sql.SparkSession

object focusDate extends App {
  val spark = SparkSession.builder.appName("Data Pre-Processing").master("local").getOrCreate()
  val sContext = spark.sparkContext
  val crimeDataFrame = spark.read
    .option("header", true)
    .option("inferSchema", true)
    .csv(StaticTool.DATA_PATH + "originCrimeData.csv")

  var dateNeedColumn = crimeDataFrame.select("Date", "Primary Type", "Year")
  dateNeedColumn.show(3)
  dateNeedColumn.printSchema()
}
```

```
+--------------------+-------------------+----+
|                Date|       Primary Type|Year|
+--------------------+-------------------+----+
|01/01/2001 11:00:...| DECEPTIVE PRACTICE|2001|
|10/08/2017 03:00:...|CRIM SEXUAL ASSAULT|2017|
|03/28/2017 02:00:...|           BURGLARY|2017|
+--------------------+-------------------+----+
only showing top 3 rows

root
 |-- Date: string (nullable = true)
 |-- Primary Type: string (nullable = true)
 |-- Year: integer (nullable = true)
```

看到Date字段居然是`String`类型. 做个cast

`foucsDate.scala` - 2

```scala
dateNeedColumn = dateNeedColumn
    .withColumn("TimeStamp", unix_timestamp(
      col("Date"), "MM/dd/yyyy HH:mm:ss").cast("timestamp"))
    .drop("Date")
    .withColumnRenamed("Primary Type", "Crime")

  dateNeedColumn.show(3)
  dateNeedColumn.printSchema()
```

```
+-------------------+----+-------------------+
|              Crime|Year|          TimeStamp|
+-------------------+----+-------------------+
| DECEPTIVE PRACTICE|2001|2001-01-01 11:00:00|
|CRIM SEXUAL ASSAULT|2017|2017-10-08 03:00:00|
|           BURGLARY|2017|2017-03-28 02:00:00|
+-------------------+----+-------------------+
only showing top 3 rows

root
 |-- Crime: string (nullable = true)
 |-- Year: integer (nullable = true)
 |-- TimeStamp: timestamp (nullable = true)
```

把时间都砍出来, 日期或者月份拿来做汇总, ~~时间用来后期画图?~~

`focusDate.scala` - 3

```scala
  dateNeedColumn = dateNeedColumn
    .withColumn("Year", col("Year"))
    .withColumn("Month", col("TimeStamp").substr(0, 7))
    .withColumn("Day", col("TimeStamp").substr(0, 10))
    .withColumn("Hour", col("TimeStamp").substr(11, 3))

  dateNeedColumn.show(5)
  dateNeedColumn.printSchema()
```

```scala
+-------------------+----+-------------------+-------+----------+----+
|              Crime|Year|          TimeStamp|  Month|       Day|Hour|
+-------------------+----+-------------------+-------+----------+----+
| DECEPTIVE PRACTICE|2001|2001-01-01 11:00:00|2001-01|2001-01-01|  11|
|CRIM SEXUAL ASSAULT|2017|2017-10-08 03:00:00|2017-10|2017-10-08|  03|
|           BURGLARY|2017|2017-03-28 02:00:00|2017-03|2017-03-28|  02|
|              THEFT|2017|2017-09-09 08:17:00|2017-09|2017-09-09|  08|
|CRIM SEXUAL ASSAULT|2017|2017-08-26 10:00:00|2017-08|2017-08-26|  10|
+-------------------+----+-------------------+-------+----------+----+
only showing top 5 rows

root
 |-- Crime: string (nullable = true)
 |-- Year: integer (nullable = true)
 |-- TimeStamp: timestamp (nullable = true)
 |-- Month: string (nullable = true)
 |-- Day: string (nullable = true)
 |-- Hour: string (nullable = true)
```



### Write Data to CSV

`focusDate.scala` - 4

```scala
  dateNeedColumn.write.option("header", true).csv(StaticTool.DATA_PATH + "dateDF.csv")
```



### Crime Column

提取完日期, 然后看看Crime列到底有多少种犯罪

`focusCrime.scala` - 1

```scala
package edu.zstu.mijazz.sparklearn1

import org.apache.spark.sql.SparkSession

object focusCrime extends App {
  val spark = SparkSession.builder.appName("Data Analysis").master("local").getOrCreate()
  val sContext = spark.sparkContext
  val data = spark.read
    .option("header", true)
    .option("inferSchema", true)
    .csv(StaticTool.DATE_DATA)
// 取回focusDate.scala中转储在hdfs中的数据
  var crimeColumnDataSet = data.select("Crime").distinct()
  crimeColumnDataSet.show(20)
  println(crimeColumnDataSet.count())
}
```

```
+--------------------+
|               Crime|
+--------------------+
|OFFENSE INVOLVING...|
|CRIMINAL SEXUAL A...|
|            STALKING|
|PUBLIC PEACE VIOL...|
|           OBSCENITY|
|NON-CRIMINAL (SUB...|
|               ARSON|
|   DOMESTIC VIOLENCE|
|            GAMBLING|
|   CRIMINAL TRESPASS|
|             ASSAULT|
|      NON - CRIMINAL|
|LIQUOR LAW VIOLATION|
| MOTOR VEHICLE THEFT|
|               THEFT|
|             BATTERY|
|             ROBBERY|
|            HOMICIDE|
|           RITUALISM|
|    PUBLIC INDECENCY|
+--------------------+
only showing top 20 rows

36
```

总共36种不同的犯罪类型.

#### Crime Summary(Spark SQL)

> `DataFrame`内操作也行, 抱着入门框架的心态, 硬上`Spark SQL`吧

只看总的犯罪统计, 抓个靠前的十宗罪吧

> 这里留了点代码, 到时候往Hive里面写或者往MariaDB里面写, 换到pyspark画图方便些.

`focusCrime.scala` - 2

> 注意的是: 要使用`spark sql`, `dataframe`或者`rdd`里面的东西要做成一个`View`, 就可以当成一个表做结构化查询了. 

```scala
  data.createOrReplaceTempView("t_CrimeDate")

  val eachCrimeSummary = spark.
    sql("select Crime, count(1) Occurs " +
      "from t_CrimeDate " +
      "group by Crime")
  // For Writing in CSV or Hive DB in further PySpark Usage
//  eachCrimeSummary.write.option("header", true).csv("")
  eachCrimeSummary.orderBy(desc("Occurs")).show(10)
```

```scala
+-------------------+-------+
|              Crime| Occurs|
+-------------------+-------+
|              THEFT|1522618|  # 偷窃
|            BATTERY|1321333|  # 殴打
|    CRIMINAL DAMAGE| 821509|  # 破坏(刑事)
|          NARCOTICS| 733993|  # 毒品犯罪
|            ASSAULT| 456288|  # 攻击
|      OTHER OFFENSE| 447617|  # 其他侵犯
|           BURGLARY| 406317|  # 非法入侵
|MOTOR VEHICLE THEFT| 331980|  # 盗窃车辆
| DECEPTIVE PRACTICE| 297519|  # 诈骗
|            ROBBERY| 270936|  # 抢劫
+-------------------+-------+
```

#### Monthly Summary(Spark SQL)

抓一下按月分类的, 看看数据是否有特征, 如果有特征就可以尝试后续做图.

`focusCrime.scala` - 3

```scala
  val groupByMonth = spark
    .sql("select month(Month) NaturalMonth, count(1) CrimePerMonth " +
      "from t_CrimeDate " +
      "group by NaturalMonth")
  groupByMonth.orderBy(desc("CrimePerMonth")).show(12)
```

```scala
+------------+-------------+
|NaturalMonth|CrimePerMonth|
+------------+-------------+
|           7|       675041|
|           8|       668824|
|           5|       644421|
|           6|       641529|
|           9|       625696|
|          10|       620504|
|           3|       594688|
|           4|       593116|
|           1|       568404|
|          11|       553769|
|          12|       525734|
|           2|       500547|
+------------+-------------+
```

这里就有很明显的趋势了, 年中部分的犯罪数量明显比年尾年头高.

## Prepare External Data

> 天气数据见[Weather Data Extraction](https://mijazzchan.github.io/posts/Weather-Data-Extraction/)

上次抓天气数据, 把2001年到今年, 每年的数据都抓下来了, 数据格式是`Date, High, Low`

```shell
 mijazz@lenovo  ~/pyProjects/.../weatherDataCsv   master ±✚  ls
2001.csv  2003.csv  2005.csv  2007.csv  2009.csv  2011.csv  2013.csv  2015.csv  2017.csv  2019.csv  
2002.csv  2004.csv  2006.csv  2008.csv  2010.csv  2012.csv  2014.csv  2016.csv  2018.csv  2020.csv
 mijazz@lenovo  ~/pyProjects/.../weatherDataCsv   master ±✚  echo "Date,High,Low" > ./temperature.full.csv
 mijazz@lenovo  ~/pyProjects/.../weatherDataCsv   master ±✚  cat ./*.csv >> ./temperature.full.csv
 ✘ mijazz@lenovo  ~/pyProjects/.../weatherDataCsv   master ±✚  head -n 3 ./temperature.full.csv 
Date,High,Low
2001-01-01,24,5
2001-01-02,19,5
 mijazz@lenovo  ~/pyProjects/.../weatherDataCsv   master ±✚  tail -n 3 ./temperature.full.csv 
2020-11-21,48,36
2020-11-22,47,41
2020-11-23,46,33
```

现在可以将其放到`hdfs`里, 然后尝试在`spark`里交叉补充好气温信息. 为可视化做准备.

```shell
 mijazz@lenovo  ~/pyProjects/.../weatherDataCsv   master ±✚  hdfs dfs -put ./temperature.full.csv /user/mijazz/chicagoData/temperature.full.csv
 mijazz@lenovo  ~/pyProjects/.../weatherDataCsv   master ±✚  hdfs dfs -ls /user/mijazz/chicagoData
Found 3 items
drwxr-xr-x   - mijazz supergroup          0 2020-11-30 21:16 /user/mijazz/chicagoData/dateDF.csv
-rw-r--r--   1 mijazz supergroup 1701238602 2020-11-30 15:43 /user/mijazz/chicagoData/originCrimeData.csv
-rw-r--r--   1 mijazz supergroup     123272 2020-11-30 23:37 /user/mijazz/chicagoData/temperature.full.csv
```

