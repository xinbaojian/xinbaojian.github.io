---
layout:     post
title:      "Spring Boot 学习起步 九、错误处理"
subtitle:   " \"Spring Boot\""
date:       2015-05-14 16:31:51
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> 错误处理

通过实现EmbeddedServletContainerCustomizer接口的customize方法即可实现自定义的错误处理,看下面的代码:

```java
@Configuration
@ComponentScan
@EnableAutoConfiguration
public class AppConfig extends WebMvcConfigurerAdapter implements EmbeddedServletContainerCustomizer{
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

    @Override
    public void customize(ConfigurableEmbeddedServletContainer factory) {
        factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/404"));
        factory.addErrorPages(new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/500"));
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**")
                .addResourceLocations("path to resources")
                .setCachePeriod(0);
    }

}
```
<!--more-->
上面的代码直接将HttpStatus.NOT_FOUND和HttpStatus.INTERNAL_SERVER_ERROR分别交给/404,/500去处理
```java
@Controller
public class IndexController {

	@RequestMapping("/404")
	public String page_404() {
		return "404";
	}

    @RequestMapping("/403")
    public String page_403() {
        return "403";
    }

	@RequestMapping("/500")
	public String page_500() {
		return "500";
	}

}
```
