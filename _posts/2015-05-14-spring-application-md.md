---
layout:     post
title:      "Spring Boot 学习起步 六、SpringApplication"
subtitle:   " \"Spring Boot\""
date:       2015-05-14 16:16:17
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> SpringApplication

spring boot通过调用SpringApplication.run启动应用.
```java
//Application.java
package demo;

import org.springframework.boot.SpringApplication;

public class Application {

    public static void main(String[] args) {
        SpringApplication.run(ApplicationConfig.class, args);
    }

}
```
<!--more-->
runa方法可以接受多个Java Configuration(@Configuration)作为入参
```java
//AppConfig.java
package demo;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.filter.CharacterEncodingFilter;
import org.springframework.web.servlet.DispatcherServlet;

@Configuration
@ComponentScan
@EnableAutoConfiguration
public class AppConfig {
    @Value("${characterEncoding}")
    String characterEncoding;

    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }

    @Bean
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        filter.setEncoding(characterEncoding);
        filter.setForceEncoding(true);
        return filter;
    }
}
```
```properties
#application.properties
characterEncoding = UTF-8
```
- 上面的AppConfig声明了2个bean,一个DispatcherServlet,一个CharacterEncodingFilter,characterEncodingFilter从配置文件中读取编码作为参数;
- 实际开发中可以根据项目需要定义任意多个bean,方法名会作为beanname,返回类型作为beantype;
- 也可以通过如下方式对bean进行定制:
```java
    @Bean(name="xxx")
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public Object foo(){
        return new Object();
    }
```

- 定制banner
可以通过如下方式禁用banner的显示
```java
public static void main(String[] args) {
SpringApplication app = new SpringApplication(MySpringConfiguration.class);
app.setShowBanner(false);
app.run(args);
}
```
也可以通过在classpath下添加banner.txt或者在配置文件中指定 banner.location,实现banner的定制.
