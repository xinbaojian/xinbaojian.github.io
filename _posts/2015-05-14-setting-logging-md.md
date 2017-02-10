---
layout:     post
title:      "Spring Boot 学习起步 五、日志处理"
subtitle:   " \"Spring Boot\""
date:       2015-05-14 16:14:05
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> 日志处理

默认情况下spring boot采用logback输出日志

在resources目录下添加logback.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender
        name="STDOUT"
        class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="STDOUT" />
    </root>
    <logger name="demo">
        <level value="DEBUG" />
    </logger>
</configuration>
```
<!--more-->
修改DemoController,添加日志内容:
```java
@RestController
public class DemoController {
    private static final Logger logger = LoggerFactory.getLogger(DemoController.class);

    @Value("${app.name}")
    String appName;

    @Value("${app.code}")
    String appCode;

    @RequestMapping("/appInfo")
    public String info() {
        logger.info("request for /appInfo...");
        return "name:"+appName+",code:"+appCode;
    }
}

```
访问[http://localhost:8080/appInfo](http://localhost:8080/appInfo),控制台会输出:
```shell
15:52:07.721 [http-nio-8080-exec-1] INFO  demo.DemoController - request for /appInfo...
```

可以通过配置项指定日志配置文件的位置;

这同时也意味着我们可以根据不同的profile指定不同日志配置文件,从而达到各种环境下的不同日志输出需求;

修改application.properties
```properties
#application.properties
app.name=demo
app.code=123456

spring.profiles.active=dev

logging.config=classpath:logback.xml
```
修改application-dev.properties
```properties
#application-dev.properties
app.name=demo for dev
app.code=111111

logging.config=classpath:logback-dev.xml

```
在resources目录下添加logback-dev.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender
        name="STDOUT"
        class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender
        name="LOGFILE"
        class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>log/demo.log</File>
        <rollingPolicy
            class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>log/%d{yyyy-MM-dd}_demo.log.zip</FileNamePattern>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%date [%thread] %-5level %logger{80} - %msg%n</pattern>
        </layout>
    </appender>
    <root level="info">
        <appender-ref ref="LOGFILE" />
    </root>
    <logger name="demo">
        <level value="DEBUG" />
    </logger>
</configuration>
```
启动项目,此时日志已经输出到了logback-dev.xml中配置的log/demo.log文件中了.

再次修改application.properties
```properties
#application.properties
app.name=demo
app.code=123456

#spring.profiles.active=dev

logging.config=classpath:logback.xml
```
注释掉#spring.profiles.active=dev,再次启动项目,日志又输出到控制台了.
