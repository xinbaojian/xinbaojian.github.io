---
layout:     post
title:      "Spring Boot 学习起步 一、快速搭建"
subtitle:   " \"Spring Boot\""
date:       2015-05-14 13:40:45
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> 快速搭建


### 第一步，创建项目

看看spring boot怎么以最简单的方式让程序跑起来的 而且还是一个web项目。

首先 使用eclipse jee创建一个maven工程，一般的工程就可以（我用的是quickstart），工程就叫spring-boot-sample；

### 第二步,添加Spring Boot依赖

在pom.xml中引入`spring-boot-start-parent`，spring官方的解释叫什么`stater POMs`，它可以提供dependency management，
也就是说依赖管理，应该是说工程需要依赖的jar包的管理，引入以后在申明其他dependency的时候就不需要version了，后面可以看到。

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.0.1.RELEASE</version>
</parent>
```
<!--more-->
### 第三步,添加Web工程依赖
因为我们开发的是web工程，所以需要在pom.xml中引入`spring-boot-starter-web`，
spring官方解释说`spring-boot-starter-web`包含了`spring webmvc`和`tomcat`等web开发的特性。

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### 第四步,编写controller类
真正的程序开始啦，我们需要一个启动类 然后在这个启动类中申明让spring boot自动给我们配置spring需要的配置，比如component scan等；为了可以尽快让程序跑起来，我们使用spring官方的实例：

```
@Controller
@EnableAutoConfiguration
public class App {
    @RequestMapping("/")
    @ResponseBody
    public String hello(){
      return "Hello world!";
    }

    public static void main(String[] args) {
      SpringApplication.run(App.class, args);
    }
}
```

##### 注解解释：
>  @EnableAutoConfiguration申明让spring boot自动给程序进行必要的配置；

> @Controller表示这个一个controller类；

> @RequestMapping("/") 表示通过/可以访问的方法；

> @ResponseBody 表示将结果直接返回给调用着。


### 第五步，运行项目。

##### 第一种方式
spring-boot-maven-plugin提供了直接运行项目的插件，我们可以直接mvn spring-boot:run运行。

```
<!-- Package as an executable JAR -->  
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-maven-plugin</artifactId>  
        </plugin>  
    </plugins>  
</build>  
```

> mvn spring-boot:run

##### 第二种方式

在App(Application)类中直接以`main`方法启动。(个人推荐)
