---
layout:     post
title:      "Spring Boot 学习起步 十一、监控"
subtitle:   " \"Spring Boot\""
date:       2015-05-14 16:46:08
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> 监控

作为一个为微服务而生的框架，自然少不了监控，下面看看spring boot为我们提供了什么。

首先你需要在pom.xml中引入如下模块：
```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
```
启动项目，即可通过如下如下路径健康项目：
- /autoconfig 显示autoconfig信息
- /beans 显示所有beans
- /configprops 显示所有配置项
- /dump
- /env 环境变量
- /health 健康检查
- /info
- /metrics
- /mappings 所有的@RequestMapping
- /shutdown 关闭引用，你需要配置endpoints.shutdown.enabled=true才能生效
- /trace 最近几次请求的信息
<!--more-->
###通过shell监控
首先添加spring-boot-starter-remote-shell依赖
```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-remote-shell</artifactId>
		</dependency>
```
重启应用，控制台显示访问密码：
```
Using default password for shell access: 57946cf9-22a1-4415-a87e-36c38801c502
```
用ssh登录：
```shell
utgard@utgard:~$ ssh -p 2000 user@localhost
Password authentication
Password:
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v1.1.8.RELEASE) on utgard
>

```
```shell
> help
Try one of these commands with the -h or --help switch:

NAME       DESCRIPTION
autoconfig Display auto configuration report from ApplicationContext
beans      Display beans in ApplicationContext
cron       manages the cron plugin
dashboard  a monitoring dashboard
egrep      search file(s) for lines that match a pattern
endpoint   Invoke actuator endpoints
env        display the term env
filter     a filter for a stream of map
java       various java language commands
jmx        Java Management Extensions
jul        java.util.logging commands
jvm        JVM informations
less       opposite of more
mail       interact with emails
man        format and display the on-line manual pages
metrics    Display metrics provided by Spring Boot
shell      shell related command
sleep      sleep for some time
sort       sort a map
system     vm system properties commands
thread     JVM thread commands
help       provides basic help
repl       list the repl or change the current repl

> jvm
usage: jvm [-h | --help] COMMAND [ARGS]

The most commonly used jvm commands are:
   runtime          Show JVM runtime
   classloading     Show JVM classloding
   compilation      Show JVM compilation
   nonheap          Show JVM memory non heap
   pool             Show JVM memory pool
   pools            Show JVM memory pools
   top
   heap             Show JVM memory heap
   gc               Show JVM garbage collection
   system           Show JVM operating system

> jvm gc

GARBAGE COLLECTION
name :PS Scavenge
name              value
------------------------------------
collection count  12
collection time   261
mpool name        PS Eden Space
mpool name        PS Survivor Space


name :PS MarkSweep
name              value
------------------------------------
collection count  3
collection time   456
mpool name        PS Eden Space
mpool name        PS Survivor Space
mpool name        PS Old Gen

```
更多内容请参考[官方文档](https://docs.spring.io/spring-boot/docs/1.2.0.M2/reference/htmlsingle/#production-ready)
