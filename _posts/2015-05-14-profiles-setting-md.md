---
layout:     post
title:      "Spring Boot 学习起步 四、Profiles配置"
subtitle:   " \"Spring Boot\""
date:       2015-05-14 15:57:01
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> Profiles 配置

spring boot通过profiles使得某些特定的配置只在某些条件下才会生效,比如之前提到的开发,测试,生产对应的三种不同的配置.

在resources目录下添加2个配置文件,application-dev.properties,application-test.properties,分别代表开发和测试环境下的配置项:

```
#application-dev.properties
user.name=demo for dev
user.age=111111
```

```
#application-test.properties
user.name=demo for dev
user.age=222222
```
修改application.properties

```
#application.properties
user.name=demo
user.age=123456
<!--more-->
#激活profile
spring.profiles.active=dev
```

启动应用,访问[http://localhost:8080/info](http://localhost:8080/info),显示:

也可以通过命令行设置profiles,多个profile用逗号分隔:

    java -jar com.baojian-1.0-SNAPSHOT.jar  --spring.profiles.active=dev,hsqldb


在Java代码中则可以通过@Profile 注解使得某些配置只在profile被激活时才生效:

```Java
@Configuration
@Profile("production")
public class ProductionConfiguration {
  // ...
}
```
