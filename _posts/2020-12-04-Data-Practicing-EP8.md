---
title: Data Practicing-EP8
author: MijazzChan
date: 2020-12-4 00:47:53 +0800
categories: [Data, Pandas]
tags: [data, python, pandas, visualization]
---

# Data Practicing-EP8

基于日期的话, 因为有index的缘故, 按日分类和按月分类都较为方便.


```python
days = ['Mon','Tue','Wed', 'Thur', 'Fri', 'Sat', 'Sun']
fullData.groupby([fullData.index.dayofweek]).size().plot(kind='barh')
plt.yticks(np.arange(7), days)
plt.xlabel('Crime Counts')
plt.show()
```


​    
![png](/assets/img/blog/20201203/output_23_0.png)
​    


周五的贡献由为突出


```python
fullData.groupby([fullData.index.month]).size().plot(kind='barh')
plt.ylabel('Month')
plt.xlabel('Crime Counts')
plt.show()
```


​    
![png](/assets/img/blog/20201203/output_25_0.png)
​    


按月分类可以看到主要集中于夏季.

> 在EP7里, `Location Description`已经被减少至20个, 排名靠后的被修改成`OTHERS`了

对犯罪发生地点的归类.


```python
fullData.groupby([fullData['Location Description']]).size().sort_values(ascending=True).plot(kind='barh')
plt.ylabel('Crime Location')
plt.xlabel('Crimes Count')
plt.show()
```


![png](/assets/img/blog/20201203/output_28_0.png)
​    


排除`OTHER`的话, 可以看到一下几个地点的犯罪发生率明显高于其它.

+ 街道
+ 居民住宅区
+ 公寓
+ 人行道

引入依赖包以及参考的缩放函数, 作多元的数据透视图以寻找数据联系. 

[Colormaps - matplotlib docs](https://matplotlib.org/tutorials/colors/colormaps.html)


```python
from sklearn.cluster import AgglomerativeClustering as AC

def scale_df(df,axis=0):
    '''
    A utility function to scale numerical values (z-scale) to have a mean of zero
    and a unit variance.
    '''
    return (df - df.mean(axis=axis)) / df.std(axis=axis)

def plot_hmap(df, ix=None, cmap='seismic', xColumn=False):
    '''
    A function to plot heatmaps that show temporal patterns
    '''
    if ix is None:
        ix = np.arange(df.shape[0])
    plt.imshow(df.iloc[ix,:], cmap=cmap)
    plt.colorbar(fraction=0.03)
    plt.yticks(np.arange(df.shape[0]), df.index[ix])
    if(xColumn):
        plt.xticks(np.arange(df.shape[1]), df.columns, rotation='vertical')
    else:
        plt.xticks(np.arange(df.shape[1]))
    plt.grid(False)
    plt.show()
    
def scale_and_plot(df, ix = None,  xCol=False):
    '''
    A wrapper function to calculate the scaled values within each row of df and plot_hmap
    '''
    df_marginal_scaled = scale_df(df.T).T
    if ix is None:
        ix = AC(4).fit(df_marginal_scaled).labels_.argsort() # a trick to make better heatmaps
    cap = np.min([np.max(df_marginal_scaled.to_numpy()), np.abs(np.min(df_marginal_scaled.to_numpy()))])
    df_marginal_scaled = np.clip(df_marginal_scaled, -1*cap, cap)
    plot_hmap(df_marginal_scaled, ix=ix, xColumn=xCol)

```

+ 犯罪发生具体时间 与 位置
+ 犯罪发生具体时间 与 犯罪类型
+ 工作日/周末     与 位置
+ 工作日/周末     与 犯罪类型
+ 位置           与 犯罪类型


```python
hour_by_location = fullData.pivot_table(values='ID', index='Location Description', columns=fullData.index.hour, aggfunc=np.size).fillna(0)
hour_by_type     = fullData.pivot_table(values='ID', index='Primary Type', columns=fullData.index.hour, aggfunc=np.size).fillna(0)
dayofweek_by_location = fullData.pivot_table(values='ID', index='Location Description', columns=fullData.index.dayofweek, aggfunc=np.size).fillna(0)
dayofweek_by_type = fullData.pivot_table(values='ID', index='Primary Type', columns=fullData.index.dayofweek, aggfunc=np.size).fillna(0)
location_by_type  = fullData.pivot_table(values='ID', index='Location Description', columns='Primary Type', aggfunc=np.size).fillna(0)
```


```python
scale_and_plot(hour_by_location)
```


​    
![png](/assets/img/blog/20201203/output_34_0.png)
​    


观察到有几块热区

+ 小巷(ALLEY), 人行道(SIDEWALK), 街道(STREET), 私家车(VEHICLE NON-COM..), 加油站(GAS STATION), 停车场/区(..PARKING LOT..)区域, 都于17点过后至午夜1点犯罪活跃.

+ 停车场(PARKING LOT..),写字楼/商业区(COMMERCIAL/BUSINESS OFFICE),学校/公共类楼宇(SCHOOL, PUBLIC BUILDING)均于早上8点至下午3点犯罪活跃.

+ 几类商店(.. STORE)均于整个中午和下午犯罪活跃. 

+ 居民住宅区和公寓型住宅均于 正午12点与午夜0点有热区

凌晨2-6时均为冷区. 这些结论也基本与常识理解较为贴近.


```python
scale_and_plot(hour_by_type)
```


​    
![png](/assets/img/blog/20201203/output_36_0.png)
​    


+ 人身侵犯, 性侵犯在中午过后几小时有热区

+ 诈骗, 欺凌类型犯罪在上午和中午有热区

其余犯罪均在18点过后, 即非工作时间存在热区. 


```python
scale_and_plot(dayofweek_by_location)
```


​    
![png](/assets/img/blog/20201203/output_38_0.png)
​    


工作日热区:

+ 停车场, 写字楼, 商店, 学校

周五/周六热区:

+ 小巷, 人行道, 餐厅, 商业区停车场, 街道, 私家车, 住宅区, 住宅区走廊/门廊.

周六周日热区:

+ 公寓, 油站, 住宅前/后院


```python
scale_and_plot(dayofweek_by_type)
```


​    
![png](/assets/img/blog/20201203/output_40_0.png)
​    


工作日热区: 

+ 人身侵犯

+ 性骚扰

+ 欺凌

+ 非法入侵

+ 卖淫

节假日热区: 

+ 妨碍公务

+ 破坏(刑事破坏)

+ 性侵犯

+ 殴打/斗殴

值得注意的是, 周五当天有几个明显的热区

+ 赌博

+ 妨碍公共安全秩序

+ 偷窃机动车

+ 青少年犯罪

+ 酒水买卖犯罪

+ 枪械犯罪


```python
scale_and_plot(location_by_type, xCol=True)
```


​    
![png](/assets/img/blog/20201203/output_42_0.png)
​    


这个犯罪地点X犯罪类型的图, 每个交叉的热点都是某种特定的犯罪形式最可能发生的地点, 这里不做赘述.

只提两个点. 

+ 斗殴和盗窃在所有地点都几乎是热区

+ 加油站地点, 几乎无赌博, 性侵, 卖淫, 酒水买卖犯罪