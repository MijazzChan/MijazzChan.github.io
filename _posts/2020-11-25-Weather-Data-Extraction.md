---
title: Weather Data Extraction
author: MijazzChan
date: 2020-11-25 15:11:32 +0800
categories: [Data, Python]
tags: [data, python, crawl]
---

# 天气数据抓取

---

> 数据来自于`National Weather Service Forecast Office`
>
> 数据服务于Hadoop+Spark Data Practicing

## 需求
需要分析的主数据来自`2001-2020`年`Chicago Area`, 故对以下年份数据进行抓取

> Data Copyright Notice:[National Weather Service Disclaimer](https://www.weather.gov/disclaimer)

## Approach
从`chrome`拿到异步请求数据的`cURL`
```shell
curl 'https://data.rcc-acis.org/StnData' \
  -H 'Connection: keep-alive' \
  -H 'Accept: application/json, text/javascript, */*; q=0.01' \
  -H 'DNT: 1' \
  -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.66 Safari/537.36' \
  -H 'Content-Type: application/x-www-form-urlencoded; charset=UTF-8' \
  -H 'Origin: https://nowdata.rcc-acis.org' \
  -H 'Sec-Fetch-Site: same-site' \
  -H 'Sec-Fetch-Mode: cors' \
  -H 'Sec-Fetch-Dest: empty' \
  -H 'Referer: https://nowdata.rcc-acis.org/' \
  -H 'Accept-Language: en,zh-CN;q=0.9,zh;q=0.8,en-XA;q=0.7' \
  --data-raw 'params=%7B%22elems%22%3A%5B%7B%22name%22%3A%22maxt%22%7D%2C%7B%22name%22%3A%22mint%22%7D%2C%7B%22name%22%3A%22maxt%22%2C%22duration%22%3A%22dly%22%2C%22normal%22%3A%221%22%2C%22prec%22%3A1%7D%2C%7B%22name%22%3A%22mint%22%2C%22duration%22%3A%22dly%22%2C%22normal%22%3A%221%22%2C%22prec%22%3A1%7D%5D%2C%22sid%22%3A%22ORDthr+9%22%2C%22sDate%22%3A%222001-01-01%22%2C%22eDate%22%3A%222001-12-31%22%7D&output=json' \
  --compressed
```
做一下处理变成python的`request`, 并且去掉一些请求参数, 只保留实际测量出的温度, 去除历史记录最高温等不相关的数据.
```python
import requests

headers = {
    'Connection': 'keep-alive',
    'Accept': 'application/json, text/javascript, */*; q=0.01',
    'DNT': '1',
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.66 Safari/537.36',
    'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
    'Origin': 'https://nowdata.rcc-acis.org',
    'Sec-Fetch-Site': 'same-site',
    'Sec-Fetch-Mode': 'cors',
    'Sec-Fetch-Dest': 'empty',
    'Referer': 'https://nowdata.rcc-acis.org/',
    'Accept-Language': 'en,zh-CN;q=0.9,zh;q=0.8,en-XA;q=0.7',
}

data = {
  'params': '{"elems":[{"name":"maxt"},{"name":"mint"}],"sid":"ORDthr 9","sDate":"2001-01-01","eDate":"2001-12-31"}',
  'output': 'json'
}

response = requests.post('https://data.rcc-acis.org/StnData', headers=headers, data=data)
print(response.content)
```

节选一点`response.content`

```shell
b'{"meta":{"state": "IL", "sids": ["ORDthr 9"], "uid": 32819, "name": "Chicago Area"},\n"data":[["2001-01-01","24","5"],\n["2001-01-02","19","5"],\n["2001-01-03","28","7"],\n["2001-01-04","30","19"],\n["2001-01-05","36","21"],\n["2001-01-06","33","17"],\n["2001-01-07","34","21"],\n["2001-01-08","26","12"],
```

可以看到`data`类下包含需要的数据

针对`post data`修改参数, 从而获得2001-2020年的数据并转成`csv`

> [Github: weatherfetch.py](https://github.com/MijazzChan/DataMiningAssignment/AssignmentFinal/weatherfetch.py)

```python
import os
import requests

headers = {
    'Connection': 'keep-alive',
    'Accept': 'application/json, text/javascript, */*; q=0.01',
    'DNT': '1',
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.66 Safari/537.36',
    'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
    'Origin': 'https://nowdata.rcc-acis.org',
    'Sec-Fetch-Site': 'same-site',
    'Sec-Fetch-Mode': 'cors',
    'Sec-Fetch-Dest': 'empty',
    'Referer': 'https://nowdata.rcc-acis.org/',
    'Accept-Language': 'en,zh-CN;q=0.9,zh;q=0.8,en-XA;q=0.7',
}


def buildPostData(year: int):
    year = str(year)
    data = {
        'params': '{"elems":[{"name":"maxt"},{"name":"mint"}],"sid":"ORDthr 9","sDate":"' + year + '-01-01","eDate":"' + year + '-12-31"}',
        'output': 'json'
    }
    return data


for year in range(2001, 2021):
    path = './weatherDataCsv'
    response = requests.post('https://data.rcc-acis.org/StnData', headers=headers, data=buildPostData(year))
    # print(response.json()["data"])
    dataList = list(response.json()["data"])
    if not os.path.isdir(path):
        os.mkdir(path)
    file = open('{}/{}.csv'.format(path, year), 'w+', encoding='utf-8')
    for eachDay in dataList:
        # Avoid Annoying '' in str output
        file.write(','.join(eachDay))
        file.write('\n')
    file.flush()
    file.close()
```

## Result
数据位于`./weatherDataCsv/*.csv`下

节选`2001.csv`

数据形式为`Date, Highest Temp(F), Lowest Temp(F)`
```csv
2001-01-01,24,5
2001-01-02,19,5
2001-01-03,28,7
2001-01-04,30,19
2001-01-05,36,21
2001-01-06,33,17
```