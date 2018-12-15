---
layout:     post
title:      "SpringBoot-事件监听的4种实现方式"
subtitle:   "\"事件-监听\"" 
date:       2018-12-14 17:03:15
author:     "xin.bj"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

Web网站如何实现单点登录，账户只能在一处登录。

首先，我们要判断服务器session集合中是否已经存在了一个session，记录该用户的登录信息。
我们可以通过HttpSessionListener监听器和全局静态map自己实现一个SessionContext。
<!--more-->

### 方式一: 手工向ApplicationContext中添加监听器

定义事件



```java
package xyz.xinbaojian.admin.event;

import lombok.Data;
import org.springframework.context.ApplicationEvent;

/**
 * @author xin.bj
 * @program micro-parent
 * @description 黑名单监听器
 * @create 2018-12-14 16:50
 **/
@Data
public class BlackListEvent extends ApplicationEvent {

    private String address;

    public BlackListEvent(Object source, String address) {
        super(source);
        this.address = address;
    }
}
```

定义监听器

```java
package xyz.xinbaojian.admin.event;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.ApplicationListener;

/**
 * @author xin.bj
 * @program micro-parent
 * @description 黑名单监听器
 * @create 2018-12-14 16:51
 **/
@Slf4j
public class BlackListListener implements ApplicationListener<BlackListEvent> {

    @Override
    public void onApplicationEvent(BlackListEvent event) {
        log.info(String.format("监听到BlackListEvent事件: %s", event.getAddress()));
        //Do Something
    }
}

```

注册监听器

```java
package xyz.xinbaojian;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import xyz.xinbaojian.admin.event.BlackListListener;

/**
 * *@author xin.bj
 * @EnableDiscoveryClient
 * //@EnableDiscoveryClient
 */
@SpringBootApplication(scanBasePackages="xyz.xinbaojian")
public class Application {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Application.class,args);
        context.addApplicationListener(new BlackListListener());
    }
}
```

### 方式二: 将监听器装载入spring容器

事件定义，定义监听器同上.
注册监听器使用注解 `@Component` 把监听器注入到Spring容器中

```java
package xyz.xinbaojian.admin.event;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

/**
 * @author xin.bj
 * @program micro-parent
 * @description 黑名单监听器
 * @create 2018-12-14 16:51
 **/
@Slf4j
@Component
public class BlackListListener implements ApplicationListener<BlackListEvent> {

    @Override
    public void onApplicationEvent(BlackListEvent event) {
        log.info(String.format("监听到BlackListEvent事件: %s", event.getAddress()));
        //Do Something
    }
}

```

### 方式三： 在application.properties中配置监听器

事件定义，定义监听器同上.

注册监听器在application.properties中配置监听

    
    context.listener.classes=xyz.xinbaojian.admin.event.BlackListListener

### 方式四： 通过@EventListener注解实现事件监听

事件定义，定义监听器同上.

注册监听器无需实现ApplicationListener接口，使用@EventListener装饰具体方法。

```java
package xyz.xinbaojian.admin.event;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

/**
 * @author xin.bj
 * @program micro-parent
 * @description 黑名单监听器
 * @create 2018-12-14 16:51
 **/
@Slf4j
@Component
public class BlackListListener {

    @EventListener
    public void onApplicationEvent(BlackListEvent event) {
        log.info(String.format("监听到BlackListEvent事件: %s", event.getAddress()));
        //Do Something
    }
}
```
