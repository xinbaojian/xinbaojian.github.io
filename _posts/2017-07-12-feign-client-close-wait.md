---
layout:     post
title:      "Spring Cloud 使用FeignClient出现Socket Time out 异常"
subtitle:   " \"Spring Cloud\""
date:       2017-07-12 11:10:22
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Cloud
---

昨天对项目进行压力测试，出现了服务无法调用的问题.

执行 `netstat -np | grep tcp` 

发现出现了几乎上千个 CLOSE_WAIT 状态的 TCP 链接。

收到的解决方案，

method.setRequestHeader("Connection", "close");


[参考资料](https://doc.nuxeo.com/blog/using-httpclient-properly-avoid-closewait-tcp-connections/)