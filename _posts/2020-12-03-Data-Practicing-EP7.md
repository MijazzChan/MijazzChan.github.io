---
title: Data Practicing-EP7
author: MijazzChan
date: 2020-12-3 20:57:01 +0800
categories: [Data, Pandas]
tags: [data, python, pandas, visualization]
---

# Data Practicing-EP7

## Visualization in Python

Pandas和notebook一起用, 在这个先被`Spark`处理过的几百万行的数据集上做可视化还是感觉方便些.

先做个依赖导入和数据清洗吧


```python
# -*- coding: utf-8 -*-
# Python Version == 3.8.6
import os
import pandas as pd
import numpy as np
from matplotlib import pyplot as plt
import seaborn as sns

plt.rcParams['figure.dpi'] = 150
plt.rcParams['savefig.dpi'] = 150
sns.set(rc={"figure.dpi": 150, 'savefig.dpi': 150})
from jupyterthemes import jtplot

jtplot.style(theme='monokai', context='notebook', ticks=True, grid=False)

fullData = pd.read_csv('~/devEnvs/chicagoCrimeData.csv', encoding='utf-8')

fullData.info()
```

    /home/mijazz/devEnvs/pyvenv/lib/python3.8/site-packages/IPython/core/interactiveshell.py:3062: DtypeWarning: Columns (21) have mixed types.Specify dtype option on import or set low_memory=False.
      has_raised = await self.run_ast_nodes(code_ast.body, cell_name,


    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 7212273 entries, 0 to 7212272
    Data columns (total 22 columns):
     #   Column                Dtype  
    ---  ------                -----  
     0   ID                    int64  
     1   Case Number           object 
     2   Date                  object 
     3   Block                 object 
     4   IUCR                  object 
     5   Primary Type          object 
     6   Description           object 
     7   Location Description  object 
     8   Arrest                bool   
     9   Domestic              bool   
     10  Beat                  int64  
     11  District              float64
     12  Ward                  float64
     13  Community Area        float64
     14  FBI Code              object 
     15  X Coordinate          float64
     16  Y Coordinate          float64
     17  Year                  int64  
     18  Updated On            object 
     19  Latitude              float64
     20  Longitude             float64
     21  Location              object 
    dtypes: bool(2), float64(7), int64(3), object(10)
    memory usage: 1.1+ GB



```python
fullData.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>Case Number</th>
      <th>Date</th>
      <th>Block</th>
      <th>IUCR</th>
      <th>Primary Type</th>
      <th>Description</th>
      <th>Location Description</th>
      <th>Arrest</th>
      <th>Domestic</th>
      <th>...</th>
      <th>Ward</th>
      <th>Community Area</th>
      <th>FBI Code</th>
      <th>X Coordinate</th>
      <th>Y Coordinate</th>
      <th>Year</th>
      <th>Updated On</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Location</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11034701</td>
      <td>JA366925</td>
      <td>01/01/2001 11:00:00 AM</td>
      <td>016XX E 86TH PL</td>
      <td>1153</td>
      <td>DECEPTIVE PRACTICE</td>
      <td>FINANCIAL IDENTITY THEFT OVER $ 300</td>
      <td>RESIDENCE</td>
      <td>False</td>
      <td>False</td>
      <td>...</td>
      <td>8.0</td>
      <td>45.0</td>
      <td>11</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2001</td>
      <td>08/05/2017 03:50:08 PM</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11227287</td>
      <td>JB147188</td>
      <td>10/08/2017 03:00:00 AM</td>
      <td>092XX S RACINE AVE</td>
      <td>0281</td>
      <td>CRIM SEXUAL ASSAULT</td>
      <td>NON-AGGRAVATED</td>
      <td>RESIDENCE</td>
      <td>False</td>
      <td>False</td>
      <td>...</td>
      <td>21.0</td>
      <td>73.0</td>
      <td>02</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017</td>
      <td>02/11/2018 03:57:41 PM</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11227583</td>
      <td>JB147595</td>
      <td>03/28/2017 02:00:00 PM</td>
      <td>026XX W 79TH ST</td>
      <td>0620</td>
      <td>BURGLARY</td>
      <td>UNLAWFUL ENTRY</td>
      <td>OTHER</td>
      <td>False</td>
      <td>False</td>
      <td>...</td>
      <td>18.0</td>
      <td>70.0</td>
      <td>05</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017</td>
      <td>02/11/2018 03:57:41 PM</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11227293</td>
      <td>JB147230</td>
      <td>09/09/2017 08:17:00 PM</td>
      <td>060XX S EBERHART AVE</td>
      <td>0810</td>
      <td>THEFT</td>
      <td>OVER $500</td>
      <td>RESIDENCE</td>
      <td>False</td>
      <td>False</td>
      <td>...</td>
      <td>20.0</td>
      <td>42.0</td>
      <td>06</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017</td>
      <td>02/11/2018 03:57:41 PM</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>11227634</td>
      <td>JB147599</td>
      <td>08/26/2017 10:00:00 AM</td>
      <td>001XX W RANDOLPH ST</td>
      <td>0281</td>
      <td>CRIM SEXUAL ASSAULT</td>
      <td>NON-AGGRAVATED</td>
      <td>HOTEL/MOTEL</td>
      <td>False</td>
      <td>False</td>
      <td>...</td>
      <td>42.0</td>
      <td>32.0</td>
      <td>02</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017</td>
      <td>02/11/2018 03:57:41 PM</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>




```python
fullData.drop_duplicates(subset=['ID', 'Case Number'], inplace=True)
fullData.drop(['Case Number', 'IUCR','Updated On','Year', 'FBI Code', 'Beat','Ward','Community Area', 'Location'], inplace=True, axis=1)
```


```python
fullData['Location Description'].describe()
```




    count     7204883
    unique        214
    top        STREET
    freq      1874164
    Name: Location Description, dtype: object




```python
fullData['Description'].describe()
```




    count     7212273
    unique        532
    top        SIMPLE
    freq       849119
    Name: Description, dtype: object




```python
fullData['Primary Type'].describe()
```




    count     7212273
    unique         36
    top         THEFT
    freq      1522618
    Name: Primary Type, dtype: object



可以看到这三列的其中两列, `Location Description`和`Description`有许多Unique值, 我们只取数量多的, 这里只取计数为前20的作为大类以做特征分析, 其他的归为杂类. 


```python
locationDescription20Except = list(fullData['Location Description'].value_counts()[20:].index)
# 用loc把数据砍掉
fullData.loc[fullData['Location Description'].isin(locationDescription20Except), fullData.columns=='Location Description'] = 'OTHER'
```


```python
description20Except = list(fullData['Description'].value_counts()[20:].index)
# 用loc把数据砍掉
fullData.loc[fullData['Description'].isin(description20Except) , fullData.columns=='Description'] = 'OTHER'
```

之前在`spark`中已经看到犯罪数量是36种, 并且数量从2001年到现在是逐年减少的. 但是只有每年的统计, 这里尝试作做`rolling sum`. 也就是每个取样点的横坐标对应一个日期, 纵坐标对应(当前日期-364天 ~ 当天)的犯罪数量和.

先把`Date`换成`Datetime`


```python
fullData.Date = pd.to_datetime(fullData.Date, format='%m/%d/%Y %I:%M:%S %p')
```

做`Resample`要有`Index`, 日期做了cast之后就行.


```python
fullData.index = pd.DatetimeIndex(fullData.Date)
fullData.resample('D').size().rolling(365).sum().plot()
plt.xlabel('Days')
plt.ylabel('Crimes Count')
plt.show()
```


​    
![png](/assets/img/blog/20201203/output_14_0.png)
​    


可以看到`rolling sum`是在稳步减少的.

现在分犯罪种类`Primary Type`来作图.


```python
eachCrime = fullData.pivot_table('ID', aggfunc=np.size, columns='Primary Type', index=fullData.index.date, fill_value=0)
eachCrime.index = pd.DatetimeIndex(eachCrime.index)
tmp = eachCrime.rolling(365).sum().plot(figsize=(12, 60), subplots=True, layout=(-1, 2), sharex=False, sharey=False)
```


​    
![png](/assets/img/blog/20201203/output_16_0.png)
​    


这里看到了一些无用的数据, 有些犯罪种类甚至近20年来发生不超过千次, 砍掉犯罪数量非前20的犯罪种类, 只留下前20的种类再做一个`rolling sum`. 

并且留意到`NON-CRIMINAL`和`NON - CRIMINAL`两个类重复, 砍掉. 并也将其变为`OTHER`


```python
crime20Except = list(fullData['Primary Type'].value_counts()[20:].index)
fullData.loc[fullData['Primary Type'].isin(crime20Except), fullData.columns=='Primary Type'] = 'OTHER'
```


```python
fullData.loc[fullData['Primary Type'] == 'NON-CRIMINAL', fullData.columns=='Primary Type'] = 'OTHER'
fullData.loc[fullData['Primary Type'] == 'NON - CRIMINAL', fullData.columns=='Primary Type'] = 'OTHER'
```


```python
eachCrime = fullData.pivot_table('ID', aggfunc=np.size, columns='Primary Type', index=fullData.index.date, fill_value=0)
eachCrime.index = pd.DatetimeIndex(eachCrime.index)
tmp = eachCrime.rolling(365).sum().plot(figsize=(12, 60), subplots=True, layout=(-1, 2), sharex=False, sharey=False)
```


​    
![png](/assets/img/blog/20201203/output_20_0.png)
​    


数据处理完之后, 明显能够看出来, 基本的犯罪种类的数量的确是在下降的, 但是有两个`WEAPONS VIOLATION`和`INTERFERENCE WITH PUBLIC OFFICER`在逆势上涨.
