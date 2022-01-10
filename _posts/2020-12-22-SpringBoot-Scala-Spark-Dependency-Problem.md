---
title: SpringBoot Scala Spark Co-Develop Dependency Problem
author: MijazzChan
date: 2020-12-22 14:33:12 +0800
categories: [Java, Tracing]
tags: [trace, error, scala, spark]
---

## Key Word

java.lang.NoClassDefFoundError: org/codehaus/janino/InternalCompilerExceptionProblem

## Description

Versions Overview

+ Java - OpenJDK8

+ Spark - 2.4.7
+ Spring Boot - 2.4.1 (Dependency Handled By pom.xml gen by start.spring.io)
+ Scala - 2.11.12
+ ~~Spark is on Hadoop YARN (Not really related)~~

Problem occurred when I tried to call scala module in Spring Boot Project to execute spark related stuff. In my case, I invoke SparkSession to read csv from my local-storage.

Main Exception Seems to be:

```java
java.lang.NoClassDefFoundError: org/codehaus/janino/InternalCompilerException
```

## Tracing 

Found in Maven project-wise `pom.xml`

```xml
<!-- Spark Dependencies Start Here -->
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
```

Artifact: `spark-sql` bring the wrong version(higher version) of `janino` and `commons-compiler`, which seems to be not compatible with spark 2.4.7 & scala 2.11.12.

## Solution

 Alter `pom.xml`, exclude the wrong version brought by `spark-sql` artifact. Append version-wise pom of `janino` and `common-compiler`.

Version `3.0.8` works. If you are using the same version of spark & scala.

Attempt to change versions to any- lower than `3.0.10` if you have different stack of versions.

```xml
<!-- https://mvnrepository.com/artifact/org.apache.spark/spark-sql -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.11</artifactId>
            <version>2.4.7</version>
            <exclusions>
                <exclusion>
                    <artifactId>janino</artifactId>
                    <groupId>org.codehaus.janino</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>commons-compiler</artifactId>
                    <groupId>org.codehaus.janino</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <artifactId>janino</artifactId>
            <groupId>org.codehaus.janino</groupId>
            <version>3.0.8</version>
        </dependency>
        <dependency>
            <artifactId>commons-compiler</artifactId>
            <groupId>org.codehaus.janino</groupId>
            <version>3.0.8</version>
        </dependency>
        <!-- Spark Dependencies End Here -->
```

mvn-repo link : [Maven Repository](https://mvnrepository.com/search?q=org.codehaus.janino)

