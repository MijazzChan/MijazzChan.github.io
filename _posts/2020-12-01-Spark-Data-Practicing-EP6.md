---
title: Spark Data Practicing-EP6
author: MijazzChan
date: 2020-12-1 18:07:28 +0800
categories: [Data, Hadoop]
tags: [data, spark, python]
---

# Spark Data Practicing-EP6

## Introduce `pyspark`

`Scala`和`Python`下对于`Spark`的操作还是有很多相似的地方的.

迁移到`PySpark`下, 因为`toPandas`和`collect() => List`这两个`pyspark`独有的特性, 使得可视化较`Scala`下方便.

不过要注意的是`Spark.DataFrame`和`Pandas.DataFrame`是两个完全不同的东西. 不过也很好理解, 鉴于这一次实验我是故意避开不使用Pandas的东西的.

假设有如下案例吧

```python
import random
def rInt():
    return random.randint(1, 100)
def rStr():
    return random.choice('I Just Dont Want To Use DataFrame From Pandas'.split(' '))
def rRow():
    return [rInt(), rStr()]

print(rRow())
```

```python
[66, 'Pandas']
[35, 'Just']
```

每次调用`rRow()`都会返回一个List, ~~也就是sparkDataFrame中的一行数据.~~

通过`Scala`中可以知道, `SparkSession`控制每次的`Spark`会话, 而他也提供一个方法来创建会话.

`parallelize()`用于`RDD`, `toDF()`会把`RDD`数据转成`Spark.DataFrame`

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder\
    .master('local').appName('Learn Pyspark').getOrCreate()

sc = spark.sparkContext
exampleSparkDataFrame = \
    sc.parallelize([rRow() for _ in range(5)]).toDF(("Number", "Word"))
exampleSparkDataFrame.show()
print(type(exampleSparkDataFrame))
```

```python
+------+---------+
|Number|     Word|
+------+---------+
|    60|DataFrame|
|    43|     Just|
|    85|     Want|
|    64|      Use|
|    52|DataFrame|
+------+---------+

<class 'pyspark.sql.dataframe.DataFrame'>
```

也可以很方便的通过`toPandas()`方式转换.

```python
examplePandasDataFrame = exampleSparkDataFrame.toPandas()
examplePandasDataFrame.info()
print(type(examplePandasDataFrame))
```

```python
RangeIndex: 5 entries, 0 to 4
Data columns (total 2 columns):
 #   Column  Non-Null Count  Dtype 
---  ------  --------------  ----- 
 0   Number  5 non-null      int64 
 1   Word    5 non-null      object
dtypes: int64(1), object(1)
memory usage: 208.0+ bytes
<class 'pandas.core.frame.DataFrame'>
```

当想取列时, `select()`选择列, `collect()`将其从远端的`Spark.DataFrame`拉回本地Python.

```python
print(exampleSparkDataFrame.select('Number').collect())
print(exampleSparkDataFrame.select('Word').collect())
```

```python
[Row(Number=6), Row(Number=16), Row(Number=50), Row(Number=53), Row(Number=51)]
[Row(Word='Just'), Row(Word='To'), Row(Word='From'), Row(Word='Just'), Row(Word='Pandas')]
```

假如你需要拿`spark.DataFrame`中的列来画图, 如下几种方法都是一样的.

```python
eg = [0 for _ in range(4)]
eg[0] = list(exampleSparkDataFrame.toPandas()['Number'])
eg[1] = exampleSparkDataFrame.select('Number').rdd.flatMap(lambda x: x).collect()
eg[2] = exampleSparkDataFrame.select('Number').rdd.map(lambda x: x[0]).collect()
eg[3] = [x[0] for x in exampleSparkDataFrame.select('Number').collect()]
for example in eg:
    print(example)
```

```python
[95, 56, 54, 61, 58]
[95, 56, 54, 61, 58]
[95, 56, 54, 61, 58]
[95, 56, 54, 61, 58]
```

但是不推荐`eg[0]`对应的方法, 他是将整个`spark.DataFrame`从远端取回来, ~~如果使用的是集群, 或者数据量比较大的话~~, 交给本地的python将其转为`Pandas.DataFrame`. 而其余几种, 而是交给spark处理过后, 单独剥离一列值进行返回.

rdd内实现的操作这里不详述.

## Start to Use PySpark

EP5中拿出了两批数据, 分别是`forPyspark.csv`和`temperature.full.csv`

先做以下导入

```python
# -*- coding: utf-8 -*-
# @Author   : MijazzChan
# Python Version == 3.8.6
import os
import pandas as pd
import numpy as np
from matplotlib import pyplot as plt
import seaborn as sns
import pylab as plot
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, round
plt.rcParams['figure.dpi'] = 150
plt.rcParams['savefig.dpi'] = 150
sns.set(rc={"figure.dpi": 150, 'savefig.dpi': 150})

DATA_PATH = "hdfs://localhost:9000/user/mijazz/chicagoData/"
```

## Something irrelevant

```python
spark = SparkSession.builder.master('local').appName('Data Visualization').getOrCreate()
weatherData = spark.read\
    .option('header', True)\
    .option('inferSchema', True)\
    .csv(DATA_PATH + 'temperature.full.csv')
# 转摄氏度
weatherData = weatherData\
    .withColumn('HighC', round((col('High').cast('float') - 32.0) / 1.8, 2))\
    .withColumn('LowC', round((col('Low').cast('float') - 32.0) / 1.8, 2))\
    .drop('High')\
    .drop('Low')

weatherData.createOrReplaceGlobalTempView('t_Weather')
weatherData.describe().show()
```

```scala
+-------+----------+------------------+------------------+
|summary|      Date|             HighC|              LowC|
+-------+----------+------------------+------------------+
|  count|      7267|              7267|              7267|
|   mean|      null|15.352508600522908| 5.617067565708001|
| stddev|      null|11.811098684239695|10.534155955862133|
|    min|2001-01-01|            -23.33|            -30.56|
|    max|2020-11-23|             39.44|             27.78|
+-------+----------+------------------+------------------+
```

拿到的数据集, `2001-01-01`年到`2020-11-23`总平均最高气温是`15.35`, 总平均最低气温是`5.62`

### Full Coverage

对着整个天气数据集画个图呢?

```python
xDays = weatherData.select('Date').rdd.flatMap(lambda x: x).collect()
yFullHigh = weatherData.select('HighC').rdd.flatMap(lambda x: x).collect()
yFullLow = weatherData.select('LowC').rdd.flatMap(lambda x: x).collect()

fig, axs = plt.subplots(2, 1)
axs[0].plot(xDays, yFullHigh)
axs[0].set_title('High Temp Full Coverage in Chicago City, 2001-2020')
axs[0].set_xlabel('Year')
axs[0].set_xticks([])
axs[0].set_ylabel('Temperature Celsius')
axs[1].plot(xDays, yFullLow)
axs[1].set_title('High Temp Full Coverage in Chicago City, 2001-2020')
axs[1].set_xlabel('Year')
axs[1].set_xticks([])
axs[1].set_ylabel('Temperature Celsius')
plt.show()
```

![plot1](/assets/img/blog/20201201/plot1.png)

仿佛看不出来什么规律. ~~说好的全球变暖呢~~

### Annual Summary

那就按年平均画个图吧

```python
annualData = \
    spark.sql('SELECT year(Date) Annual, round(avg(HighC), 2) avgHigh, round(avg(LowC), 2) avgLow ' 
          'FROM global_temp.t_Weather '
          'GROUP BY year(Date) ')\
    .orderBy(asc('Annual'))
annualData.show(20)
```

```scala
+------+-------+------+
|Annual|avgHigh|avgLow|
+------+-------+------+
|  2001|  15.39|  5.49|
|  2002|  15.37|  5.62|
|  2003|  14.63|  4.24|
|  2004|  14.98|  4.88|
|  2005|  15.87|  5.53|
|  2006|   15.9|  6.31|
|  2007|   15.6|  5.84|
|  2008|  14.25|  4.38|
|  2009|  14.05|  4.58|
|  2010|  15.66|  6.07|
|  2011|  15.04|  5.85|
|  2012|  17.73|   7.3|
|  2013|  14.43|  4.68|
|  2014|  13.66|  3.76|
|  2015|  15.02|  5.26|
|  2016|  15.97|  6.57|
|  2017|  16.27|  6.59|
|  2018|  15.12|  6.08|
|  2019|  14.44|  5.31|
|  2020|  17.91|  8.26|
+------+-------+------+
```

```python
fig, axs = plt.subplots(2, 1)
xYear = annualData.select('Annual').collect()
yAvgHigh = annualData.select('avgHigh').collect()
yAvgLow = annualData.select('avgLow').collect()

axs[0].plot(xYear, yAvgHigh)
axs[0].set_title('Average High Temp in Chicago City')
axs[0].set_xlabel('Year')
axs[0].set_ylabel('Temperature Celsius')
axs[1].plot(xYear, yAvgLow)
axs[1].set_title('Average Low Temp in Chicago City')
axs[1].set_xlabel('Year')
axs[1].set_ylabel('Temperature Celsius')
plt.show()
```

![plot2](/assets/img/blog/20201201/plot2.png)

现在是能看出一些趋势了.