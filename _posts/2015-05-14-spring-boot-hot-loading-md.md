---
layout:     post
title:      "Spring Boot 学习起步 二，支持热加载"
subtitle:   " \"Spring Boot\""
date:       2015-05-14 13:42:17
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> 支持热加载

### 使spring boot项目支持热加载
- 在STS中开发spring bootx项目，如果采用run as启动项目，修改代码后是不支持热加载的；
- 如果采用debug模式，虽然可以支持热加载，但是仍然有部分代码修改后不能生效，比如修改了@RequestMapping，@Controller，必须重启后才能生效；

### 通过Spring Loaded实现热加载
- 项目地址：[spring loaded](https://github.com/spring-projects/spring-loaded)
- 下载[springloaded-1.2.1.RELEASE.jar](http://search.maven.org/remotecontent?filepath=org/springframework/springloaded/1.2.1.RELEASE/springloaded-1.2.1.RELEASE.jar)
- 右键Application.java-->Run as-->Run Configurations
- 在Arguments页签中的VM arguments项加入如下参数，其中E:\java\springloaded-1.2.1.RELEASE.jar指向springloaded-1.2.1.RELEASE.jar的保存路径

```
-javaagent:D:\dev\jar\springloaded-1.2.1.RELEASE.jar -noverify
```

<!--more-->

- 点击run,启动完成后查看[http://localhost:8080/](http://localhost:8080/)
- 修改Controller，添加一个@RequestMapping("/test")

```
@RestController
class DemoController {
	@RequestMapping("/")
	public String hello() {
		return "Hello world!";
	}

	@RequestMapping("/test")
	public String test() {
		return "Hello world by hotload!";
	}
}
```
- 查看[http://localhost:8080/test](http://localhost:8080/test)，此时修改后的代码已经生效。
