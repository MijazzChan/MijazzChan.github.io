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