---
title: Spark Data Practicing-EP0
author: MijazzChan
date: 2020-11-25 17:04:39 +0800
categories: [Data, Hadoop]
tags: [data, hadoop, spark, java]
---

# Spark Data Practicing-EP0

## Before You Read This

该文只是一门课程`数据挖掘`的大作业记录文, 并不是教程式文章. 我会尽量详细的给出参考文与具体步骤.

## Why This Topic?

看了很多人写的技术博客, 发现许多都是基于`Pseudo-Distributed Mode`或者`Fully-Distributed Mode`. 这两种模式因为资源问题我也只成功搭建并使用过前者, 当时在`Windows10`上拖着一个`CentOS8`的虚拟机, 因为数据集就快`2GB`大小, 一套下来发现虚拟机吃的内存快接近`7G`.

~~又想试试YARN又不想做多机分布式, 那就做单机YARN~~ 关于YARN:官方的也很简明易懂 [Apache Hadoop YARN](https://hadoop.apache.org/docs/r3.1.4/hadoop-yarn/hadoop-yarn-site/YARN.html)

毕竟数据集只是几G, python直接上pandas套装估计会更方便.

抱着入门一下这个技术栈, 完成一下大作业和想拥抱一下Arch社区的心态, 在使用过`LinuxMint`和`Fedora`的我换到了`Manjaro Linux 20`. 

```shell
 mijazz@lenovo  ~  screenfetch

 ██████████████████  ████████     mijazz@lenovo
 ██████████████████  ████████     OS: Manjaro 20.2 Nibia
 ██████████████████  ████████     Kernel: x86_64 Linux 5.8.18-1-MANJARO
 ██████████████████  ████████     Uptime: 16m
 ████████            ████████     Packages: 1239
 ████████  ████████  ████████     Shell: zsh 5.8
 ████████  ████████  ████████     Resolution: 1920x1080
 ████████  ████████  ████████     DE: KDE 5.76.0 / Plasma 5.20.3
 ████████  ████████  ████████     WM: KWin
 ████████  ████████  ████████     GTK Theme: Breath [GTK2/3]
 ████████  ████████  ████████     Icon Theme: breath2-dark
 ████████  ████████  ████████     Disk: 18G / 102G (19%)
 ████████  ████████  ████████     CPU: Intel Core i5-7300HQ @ 4x 3.5GHz [45.0°C]
 ████████  ████████  ████████     GPU: GeForce GTX 1050
                                  RAM: 2880MiB / 15904MiB
 mijazz@lenovo  ~  uname -a   
Linux lenovo 5.8.18-1-MANJARO #1 SMP PREEMPT Sun Nov 1 14:10:04 UTC 2020 x86_64 GNU/Linux
```

## Prerequisite

+ Java - OpenJDK8

  > 为了避免奇怪的兼容性问题, 根据`Hadoop`官方的`wiki`, 其推荐版本是8, 并且更高版本的`Hadoop`支持11. 同时明确指明了`OpenJDK8`是官方用于其编译的版本. 此处参考: [cwiki-Apache](https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions)

  这里给出`AdoptOpenJDK8`的地址

  > Redhat的, OpenJDK自编译的, 又或者来自`Pacman`包管理的都应该不成问题.
  >
  > 但是我在`CentOS8`上首次尝试`Hadoop`的时候, `conf`里也要多配置一次`$JAVA_HOME`有点奇怪

  [Official URL](https://github.com/AdoptOpenJDK/openjdk8-binaries/releases/download/jdk8u275-b01/OpenJDK8U-jdk_x64_linux_hotspot_8u275b01.tar.gz)

  [清华tuna镜像](https://mirrors.tuna.tsinghua.edu.cn/AdoptOpenJDK/8/jdk/x64/linux/)

+ Hadoop - Hadoop 3.1.4 Tarball

  [Official URL](https://www.apache.org/dyn/closer.cgi/hadoop/common)

  [Aliyun Mirrors](https://mirrors.aliyun.com/apache/hadoop/common/hadoop-3.1.4/hadoop-3.1.4.tar.gz)

+ Spark  - Spark  2.4.7 Tarball

  选` Pre-built with user-provided Apache Hadoop ` 因采用`YARN`部署

  [Official URL](https://www.apache.org/dyn/closer.lua/spark/spark-2.4.7)

  [Aliyun Mirrors](https://mirrors.aliyun.com/apache/spark/spark-2.4.7/spark-2.4.7-bin-without-hadoop.tgz)

## Prepare Environment

### Extract JDK

```shell
mijazz@lenovo  ~/devEnvs  ll -a
total 100M
drwxr-xr-x  6 mijazz mijazz 4.0K Nov 25 15:23 .
drwx------ 36 mijazz mijazz 4.0K Nov 25 15:24 ..
-rw-r--r--  1 mijazz mijazz 100M Nov 24 15:20 OpenJDK8U-jdk_x64_linux_hotspot_8u275b01.tar.gz

mijazz@lenovo  ~/devEnvs  tar -xf ./OpenJDK8U*.tar.gz                      
mijazz@lenovo  ~/devEnvs  mv ./jdk8u275-b01 ./OpenJDK8
mijazz@lenovo  ~/devEnvs  rm ./OpenJDK8U*.tar.gz                           
mijazz@lenovo  ~/devEnvs  ll -a
total 28K
drwxr-xr-x  7 mijazz mijazz 4.0K Nov 25 15:28 .
drwx------ 36 mijazz mijazz 4.0K Nov 25 15:28 ..
drwxr-xr-x  8 mijazz mijazz 4.0K Nov  9 20:23 OpenJDK8
```

### Configure $PATH

> 我使用的是`zsh`, 并且不索引`bashrc`. 虽然到时候我会在主shell里启动, 但是不保证他会不会在某些部件里调用bash. 
>
> 我就把环境变量同时加进`~/.bashrc`和`~/.zshrc`里了.
>
> 如果你是使用`bash`, 只需加进`~/.bashrc`, 记得`source`或重启shell生效.

```shell
echo '# Java Environment Variable' >> ~/.bashrc
echo 'export JAVA_HOME="/home/mijazz/devEnvs/OpenJDK8"' >> ~/.bashrc
echo 'export JRE_HOME="${JAVA_HOME}/jre"' >> ~/.bashrc
echo 'export CLASSPATH=".:${JAVA_HOME}/lib:${JRE_HOME}/lib"' >> ~/.bashrc
echo 'export PATH="${JAVA_HOME}/bin:$PATH"' >> ~/.bashrc
# Only if u r using zsh
echo '# Java Environment Variable' >> ~/.zshrc
echo 'export JAVA_HOME="/home/mijazz/devEnvs/OpenJDK8"' >> ~/.zshrc
echo 'export JRE_HOME="${JAVA_HOME}/jre"' >> ~/.zshrc
echo 'export CLASSPATH=".:${JAVA_HOME}/lib:${JRE_HOME}/lib"' >> ~/.zshrc
echo 'export PATH="${JAVA_HOME}/bin:$PATH"' >> ~/.zshrc
```

### Verify Configuration

```shell
 mijazz@lenovo  ~  echo $JAVA_HOME
/home/mijazz/devEnvs/OpenJDK8
 mijazz@lenovo  ~  java -version  
openjdk version "1.8.0_275"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_275-b01)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.275-b01, mixed mode)
```

### SSH, sshd.service

>  [Hadoop: Setting up a Single Node Cluster.](https://hadoop.apache.org/docs/r3.1.4/hadoop-project-dist/hadoop-common/SingleCluster.html)
>
> ssh must be installed and sshd must be running to use the Hadoop scripts that manage remote Hadoop daemons if the optional start and stop scripts are to be used.

```shell
 mijazz@lenovo  ~  ssh localhost
ssh: connect to host localhost port 22: Connection refused
 mijazz@lenovo  ~  systemctl status sshd
● sshd.service - OpenSSH Daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; disabled; vendor preset: disabled)
     Active: inactive (dead)
```

明显没开, 配置一下sshd和限制root远程登录什么的, 还有只允许公匙登录什么的.

sshd_config

```
PasswordAuthentication no
PermitEmptyPasswords no
PubkeyAuthentication yes
PermitRootLogin no
```

权限记得设`0600`, Unix/Linux对这类权限要求很严格.

```shell
ssh-keygen -t rsa -b 4096
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

 mijazz@lenovo  ~  sudo systemctl start sshd 
[sudo] password for mijazz: 
 mijazz@lenovo  ~  sudo systemctl enable sshd
Created symlink /etc/systemd/system/multi-user.target.wants/sshd.service → /usr/lib/systemd/system/sshd.service.
 mijazz@lenovo  ~  ssh localhost             
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is SHA256:OvL0/qmZWaRDL66+wbprrEiK4XhNgo1FAU/jRoWIsc0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
 mijazz@lenovo  ~  exit
Connection to localhost closed.
```

公匙免密登录就配置好了



