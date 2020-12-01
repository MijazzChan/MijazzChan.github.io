---
title: Spark Data Practicing-EP6
author: MijazzChan
date: 2020-12-1 18:07:28 +0800
categories: [Data, Hadoop]
tags: [data, spark, python]
---

# Spark Data Practicing-EP6

## Merge to Python

EP5中拿出了两批数据, 分别是`forPyspark.csv`和`temperature.full.csv`

迁移到`PySpark`下, 因为`toPandas`和`collect() => List`这两个`pyspark`独有的特性, 使得可视化较`Scala`下方便.

不过要注意的是`Spark.DataFrame`和`Pandas.DataFrame`是两个完全不同的东西.

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

![plot2](/assets/img/20201201/plot2.png)

现在是能看出一些趋势了.