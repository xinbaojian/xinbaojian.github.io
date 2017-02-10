---
layout:     post
title:      "Spring Boot 学习起步 三、一些设置"
subtitle:   " \"Spring Boot\""
date:       2015-05-14 13:43:05
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> 一些设置

#### 一、指定启动类
在之前的demo项目中任意创建一个类,并添加main函数,如下所示:

```Java
package demo;
public class Foo {
    public static void main(String[] args) {
    }
}
```

然后用maven对项目进行打包

    mvn clean package

会提示如下错误:
```
Failed to execute goal org.springframework.boot:spring-boot-maven-plugin:1.2.3.RELEASE:repackage (default) on project com.baojian: Execution default of goal org.springframework.boot:spring-boot-maven-plugin:1.2.3.RELEASE:repac
kage failed: Unable to find a single main class from the following candidates [com.baojian.App, com.baojian.Foo] -> [Help 1]
```

默认情况下,spring boot会查找jar包下包含main函数的类作为程序的入口; 如果找到多个,则会报错,此时可以通过在pom.xml指定start class,从而明确告诉spring boot项目的启动类.
<!--more-->
```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <start-class>demo.Application</start-class>
    <java.version>1.7</java.version>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

这样就可以打包成功了，找到打包的jar包，执行以下命令就把这个Spring Boot的Web项目跑起来了

    java -jar com.baojian-1.0-SNAPSHOT.jar


#### 二、配置文件

- 几乎每个项目都有一些配置项,诸如数据源,缓存,系统参数,第三方接口...
- 不同的环境下配置都是不一样的,很多时候开发是一套配置,测试是另一套,生产又是一套,甚至出于安全考虑,配置项不是对所有人都可见的.
- 如果采用单一的配置文件,在不同环境中切换就需要频繁的修改,很多工作中坑爹的问题都出自这里.

Spring 是如何解决这些问题呢？

    application.properties

默认情况下,spring boot从application.properties中读取配置项

修改demo项目中resources目录下的application.properties,添加如下配置项:

```
#app config
app.name=demo
app.code=123456
```

代码中可以通过`@Value("${key}")`的方式获取对应的值

```Java
package com.baojian;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.context.web.SpringBootServletInitializer;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 用户测试读取 application.properties 文件
 *
 * “@EnableAutoConfiguration”注解的作用在于让 Spring Boot 根据应用所声明的依赖来对 Spring 框架进行自动配置，这就减少了开发人员的工作量。
 * “@RestController”和”@RequestMapping”由 Spring MVC 提供，用来创建 REST 服务。这两个注解和 Spring Boot 本身并没有关系。
 * “@ResponseBody”将内容或对象作为 HTTP 响应正文返回，并调用适合HttpMessageConverter的Adapter转换对象，写入输出流。
 * Created by xinbaojian on 2015/5/14.
 */
@RestController
@EnableAutoConfiguration
public class App extends SpringBootServletInitializer{

    @Value("${user.name}")
    private String name;

    @Value("${user.age}")
    private String age;

    @RequestMapping("/")
    public String hello(){
        return "Hello world!";
    }

    @RequestMapping("/info")
    public String userInfo(){
        String info = String.format("姓名：%s 年龄:%s",name,age);
        System.out.println("info:"+info);
        return info;
    }

    public static void main(String[] args) {
        SpringApplication.run(App.class,args);
    }
}
```
访问[http://localhost:8080/info](http://localhost:8080/info) 即可正确看到 `姓名：xinbaojian 年龄:18` 输出

#### 通过外部文件设置配置项

在应用中管理配置并不是一个容易的任务，尤其是在应用需要部署到多个环境中时。通常会需要为每个环境提供一个对应的属性文件，用来配置各自的数据库连接信息、服务器信息和第三方服务账号等。通常的应用部署会包含开发、测试和生产等若干个环境。不同的环境之间的配置存在覆盖关系。测试环境中的配置会覆盖开发环境，而生产环境中的配置会覆盖测试环境。Spring 框架本身提供了多种的方式来管理配置属性文件。Spring 3.1 之前可以使用 PropertyPlaceholderConfigurer。Spring 3.1 引入了新的环境（Environment）和概要信息（Profile）API，是一种更加灵活的处理不同环境和配置文件的方式。不过 Spring 这些配置管理方式的问题在于选择太多，让开发人员无所适从。Spring Boot 提供了一种统一的方式来管理应用的配置，允许开发人员使用属性文件、YAML 文件、环境变量和命令行参数来定义优先级不同的配置值。
Spring Boot 所提供的配置优先级顺序比较复杂。按照优先级从高到低的顺序，具体的列表如下所示。

* 1.命令行参数。
* 2.通过 System.getProperties() 获取的 Java 系统参数。
* 3.操作系统环境变量。
* 4.从 java:comp/env 得到的 JNDI 属性。
* 5.通过 RandomValuePropertySource 生成的“random.*”属性。
* 6.应用 Jar 文件之外的属性文件。
* 7.应用 Jar 文件内部的属性文件。
* 8.在应用配置 Java 类（包含“@Configuration”注解的 Java 类）中通过“@PropertySource”注解声明的属性文件。
* 9.通过“SpringApplication.setDefaultProperties”声明的默认属性。

Spring Boot 的这个配置优先级看似复杂，其实是很合理的。比如命令行参数的优先级被设置为最高。这样的好处是可以在测试或生产环境中快速地修改配置参数值，而不需要重新打包和部署应用。

SpringApplication 类默认会把以“--”开头的命令行参数转化成应用中可以使用的配置参数，如 “--name=Alex” 会设置配置参数 “name” 的值为 “Alex”。如果不需要这个功能，可以通过` “SpringApplication.setAddCommandLineProperties(false)”` 禁用解析命令行参数。

先将刚才的项目打包,然后在生产jar包的同级目录添加application.properties
```
cd spring-boot-sample
mvn clean package
cd target
echo user.name=demo from outer file > application.properties
java -jar com.baojian-1.0-SNAPSHOT.jar
```
PS: 我做测试时，外部文件无法加载覆盖内部文件，不知道什么情况，有股淡淡的忧伤。。

#### 读取位置以及优先级

默认情况下,spring boot会从如下几个地方读取application.properties:

* 命令行参数
* 当前应用同级目录下的/config目录
* 当前应用的同级目录
* classpath下的/config包
* classpath

如果存在相同的配置项,那么会按优先级覆盖,优先级按上面的顺序从高到低,即:

    命令行 > /config目录 > 当前目录 > classpath:/config > classpath
