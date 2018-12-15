---
layout:     post
title:      "Spring Boot 学习起步 八、静态资源"
subtitle:   " \"Spring Boot\""
date:       2015-05-14 16:30:15
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> 静态资源

默认情况下,spring boot会通过ResourceHttpRequestHandler从如下位置注册静态资源:
- /static目录
- /public目录
- /resources目录
- /META-INF/resources目录
- ServletContext根目录

我们可以通过继承WebMvcConfigurerAdapter,重写addResourceHandlers方法来实现定制化.
```java
@Configuration
class ClientResourcesConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/**")
				.addResourceLocations("/WEB-INF/resources/")
				.setCachePeriod(0);
	}
}
```
