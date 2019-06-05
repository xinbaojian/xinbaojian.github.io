---
layout:     post
title:      "Dockerfile 构建Java镜像"
subtitle:   "\"Docker 构建Java镜像\"" 
author:     "xin.bj"
header-img: "img/post-bg-2015.jpg"
tags:
    - Linux
    - Ubuntu
    - Docker
    - Java
---

`Dockerfile` 如下

```sehll
FROM daocloud.io/library/ubuntu:18.04

MAINTAINER  baojian

ADD jdk-8u211-linux-x64.tar.gz /usr/local/

ENV TZ=Asia/Shanghai

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN apt-get update
RUN apt-get install tzdata -y
RUN dpkg-reconfigure --frontend noninteractive tzdata

ENV JAVA_HOME /usr/local/jdk1.8.0_211
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV PATH $PATH:$JAVA_HOME/bin
```

### Dockerfile 解释

```shell
FROM daocloud.io/library/ubuntu:18.04
```
基础镜像：版本

运行以下命令（确保当前路径下只有dockerfile文件）：

```shell
docker build -t java:8-211 .
```
### Tips

    1. jdk拷贝到dockerfile同级目录，如果在其它目录拷贝的时候可能出现找不到目录错误； 
    2. 使用ADD指令会直接对jdk-8u144-linux-x64.tar.gz进行解压缩，不用再单独的tar解压jdk了。
    3. -t 指定镜像的名称和tag； 
    4. 上面有个 . ,这个表示当前目录，必不可少的