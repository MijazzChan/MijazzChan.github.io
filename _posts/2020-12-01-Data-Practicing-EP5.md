---
title: Data Practicing-EP5
author: MijazzChan
date: 2020-12-1 14:16:02 +0800
categories: [Data, Spark]
tags: [data, spark, scala]
---

# Data Practicing-EP5

## Get Weather Data

`StaticTool.java` - +(Add Row)

```java
+    public static final String WEATHER_DATA = DATA_PATH + "temperature.full.csv";
```

`MergeWeather.scala` - 1

```scala
package edu.zstu.mijazz.sparklearn1

import org.apache.spark.sql.SparkSession

object MergeWeather extends App{
  val spark = SparkSession.builder.appName("Data Analysis").master("local").getOrCreate()
  val sContext = spark.sparkContext
  val data = spark.read
    .option("header", true)
    .option("inferSchema", false)
    .csv(StaticTool.WEATHER_DATA)

  data.printSchema()
  data.show(3)

}
```

```scala
root
 |-- Date: string (nullable = true)
 |-- High: string (nullable = true)
 |-- Low: string (nullable = true)
 
+----------+----+---+
|      Date|High|Low|
+----------+----+---+
|2001-01-01|  24|  5|
|2001-01-02|  19|  5|
|2001-01-03|  28|  7|
+----------+----+---+
only showing top 3 rows
```

> `inferSchema==false` 因为在EP4`dataFrame`中有一列的`Day`的类型本就是`string`, 这里如果给了true, 就被内转成`timestamp`了, 对后续`sql`不算太方便.

## Merge them Together

在EP4中我曾经通过`createOrReplaceTempView()`来创建了一张临时表, 但是这张表的生命周期是绑定在`SparkSession`上的, 现在我换了一个`Session`, 现在采用`createGlobalTempView()`

详细见文章总结.

`focusDate.scala` - (- means delete row, + means add row)

```scala
-  data.createOrReplaceTempView("t_CrimeDate")
+  data.createOrReplaceGlobalTempView("t_CrimeDate")

  val eachCrimeSummary = spark.
    sql("select Crime, count(1) Occurs " +
-      "from t_CrimeDate " + 
+      "from global_temp.t_CrimeDate " +
      "group by Crime")

   val groupByMonth = spark
    .sql("select month(Month) NaturalMonth, count(1) CrimePerMonth " +
-      "from t_CrimeDate " +   
+      "from global_temp.t_CrimeDate " +
      "group by NaturalMonth")

```

通过前面生成了`globalTempView`之后, 就可以在另一个Session中来通过`global_temp.表名`来访问了.

这里采用spark sql来进行数据合并. 先把表做出来

`MergeWeather.scala` - 2

```scala
  data.createOrReplaceGlobalTempView("t_weatherData")
```

先看一下两张表的~~Schema关联~~长什么样

```scala
// global_temp.t_CrimeDate
root
 |-- Crime: string (nullable = true)
 |-- Year: integer (nullable = true)
 |-- TimeStamp: timestamp (nullable = true)
 |-- Month: string (nullable = true)
 |-- Day: timestamp (nullable = true)
 |-- Hour: integer (nullable = true)
 // global_temp.t_weatherData
 root
 |-- Date: string (nullable = true)
 |-- High: string (nullable = true)
 |-- Low: string (nullable = true)
```

`global_temp.t_CrimeDate.Day` <==> `global_temp.t_weatherData.Date`

`High, Low` Join上

`MergeWeather.scala` - 3

```scala
  val mergedData = spark.newSession()
    .sql("select C.Crime, C.Year, C.TimeStamp, C.Month, C.Day, W.High, W.Low C.Location " +
      "from global_temp.t_CrimeDate C, global_temp.t_weatherData W " +
      "where C.Day = W.Date")
  mergedData.printSchema()
  mergedData.show(3)
```

```scala
+-------------------+----+-------------------+-------+-------------------+----+---+---------+
|              Crime|Year|          TimeStamp|  Month|                Day|High|Low| Location|
+-------------------+----+-------------------+-------+-------------------+----+---+---------+
| DECEPTIVE PRACTICE|2001|2001-01-01 11:00:00|2001-01|2001-01-01 00:00:00|  24|  5|RESIDENCE|
|CRIM SEXUAL ASSAULT|2017|2017-10-08 03:00:00|2017-10|2017-10-08 00:00:00|  78| 54|RESIDENCE|
|           BURGLARY|2017|2017-03-28 02:00:00|2017-03|2017-03-28 00:00:00|  50| 36|    OTHER|
+-------------------+----+-------------------+-------+-------------------+----+---+---------+
only showing top 3 rows
```

所以现在每条犯罪记录都有了当天的天气信息了.

但是温标是华氏温标, `(F - 32) / 1.8 = C`. 用`DataFrame`来做就行, ~~虽然当时SQL导入的时候也可以这样做.~~

`MergeWeather.scala` - 4

```scala
  mergedData = mergedData
    .withColumn("HighC", round(col("High").cast("float").-(32.0)./(1.8), 2))
    .withColumn("LowC", round(col("Low").cast("float").-(32.0)./(1.8), 2))
    .drop("High")
    .drop("Low")

  mergedData.printSchema()
  mergedData.show(3)
```

```scala
+-------------------+----+-------------------+-------+-------------------+---------+-----+-----+
|              Crime|Year|          TimeStamp|  Month|                Day| Location|HighC| LowC|
+-------------------+----+-------------------+-------+-------------------+---------+-----+-----+
| DECEPTIVE PRACTICE|2001|2001-01-01 11:00:00|2001-01|2001-01-01 00:00:00|RESIDENCE|-4.44|-15.0|
|CRIM SEXUAL ASSAULT|2017|2017-10-08 03:00:00|2017-10|2017-10-08 00:00:00|RESIDENCE|25.56|12.22|
|           BURGLARY|2017|2017-03-28 02:00:00|2017-03|2017-03-28 00:00:00|    OTHER| 10.0| 2.22|
+-------------------+----+-------------------+-------+-------------------+---------+-----+-----+
only showing top 3 rows
```

`MergeWeather.scala` - 5

> reparition(1) 
>
> Returns a new Dataset partitioned by the given partitioning expressions into `numPartitions`. The resulting Dataset is hash partitioned.

```
  mergedData
    .repartition(1)
    .write
    .option("header", true)
    .csv(StaticTool.DATA_PATH + "forPySpark.csv")
```

