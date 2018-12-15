---
layout:     post
title:      "ReadMore Test"
subtitle:   "\"事件-监听\"" 
date:       2018-12-15 10:03:15
author:     "xin.bj"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

从权限控制的设计流程上说，实线代表具体权限操作流程，虚线代表附加业务流程和结果流程，椭圆形文本框表示和RBAC权限控制模型（见图1）的对应环节，整个权限控制流程描述如下：

1）用户登录系统申请权限，申请权限流程流转至系统管理员处，登录日志信息流转至安全审计员处；

2）系统管理员收到申请权限请求后，对用户和角色进行登记，并进行角色拟分配，同时完成角色功能表和分配表的初步制定。授权数据完成后交付至安全保密管理员处，授权过程中将产生日志推送至安全审计员处；

3）安全保密员收到授权数据后，对数据进行复核和审批。无论是否通过，审批结果均反馈给系统管理员。只有通过审批后，授权数据才真正成为有效的授权，同时通过审批的授权流程将生成操作日志提交至安全审计员处；

4）系统管理员收到安全保密员的审批反馈结果后，对用户进行赋权操作：通过则进行授权，进行相应权限的分配，不通过则拒绝授权，用户申请的权限不被批准；

5）安全审计员将全程监督整个授权过程，包括对登录日志、推送日志和操作日志的审计和备查，并可导出需要的日志数据，全程监督安全保密员和系统管理员的操作过程。

与普通RBAC权限控制模型相比较，三员分立权限控制模型具有以下优点：

1）在权限设置上采用了三员分立的模式，即在权限分配过程中赋予权、审批权和审计权三者之间分离。三个权限拥有者之间是相互独立、相互制约的，从而有效解决了系统管理员权限过大的问题；



从权限控制的设计流程上说，实线代表具体权限操作流程，虚线代表附加业务流程和结果流程，椭圆形文本框表示和RBAC权限控制模型（见图1）的对应环节，整个权限控制流程描述如下：

1）用户登录系统申请权限，申请权限流程流转至系统管理员处，登录日志信息流转至安全审计员处；

2）系统管理员收到申请权限请求后，对用户和角色进行登记，并进行角色拟分配，同时完成角色功能表和分配表的初步制定。授权数据完成后交付至安全保密管理员处，授权过程中将产生日志推送至安全审计员处；

3）安全保密员收到授权数据后，对数据进行复核和审批。无论是否通过，审批结果均反馈给系统管理员。只有通过审批后，授权数据才真正成为有效的授权，同时通过审批的授权流程将生成操作日志提交至安全审计员处；

4）系统管理员收到安全保密员的审批反馈结果后，对用户进行赋权操作：通过则进行授权，进行相应权限的分配，不通过则拒绝授权，用户申请的权限不被批准；

5）安全审计员将全程监督整个授权过程，包括对登录日志、推送日志和操作日志的审计和备查，并可导出需要的日志数据，全程监督安全保密员和系统管理员的操作过程。

与普通RBAC权限控制模型相比较，三员分立权限控制模型具有以下优点：

1）在权限设置上采用了三员分立的模式，即在权限分配过程中赋予权、审批权和审计权三者之间分离。三个权限拥有者之间是相互独立、相互制约的，从而有效解决了系统管理员权限过大的问题；