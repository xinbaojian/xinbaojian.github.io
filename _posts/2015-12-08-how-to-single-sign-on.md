---
layout:     post
title:      "单点登录实现"
subtitle:   " \"Spring Boot\""
date:       2015-12-08 10:48:24
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> 单点登录

Web网站如何实现单点登录，账户只能在一处登录。

首先，我们要判断服务器session集合中是否已经存在了一个session，记录该用户的登录信息。
我们可以通过HttpSessionListener监听器和全局静态map自己实现一个SessionContext。
<!--more-->

MySessionContext.java
```
package com.vcooline.wshop.controller.base;

import java.util.HashMap;

import javax.servlet.http.HttpSession;

public class MySessionContext {

    private static MySessionContext instance;
    private HashMap<String, HttpSession> mymap;

    private MySessionContext() {
        mymap = new HashMap<String, HttpSession>();
    }

    public static MySessionContext getInstance() {
        if (instance == null) {
            instance = new MySessionContext();
        }
        return instance;
    }

    public synchronized void AddSession(HttpSession session) {
        if (session != null) {
            mymap.put(session.getId(), session);
        }
    }

    public synchronized void DelSession(HttpSession session) {
        if (session != null) {
            mymap.remove(session.getId());
        }
    }

    public synchronized HttpSession getSession(String session_id) {
        if (session_id == null) return null;
        return (HttpSession) mymap.get(session_id);
    }

    public synchronized HashMap<String, HttpSession> getMap() {
        return mymap;
    }
}

```

LoginSessionListener.java

```
package com.vcooline.wshop.listener;

import com.vcooline.wshop.controller.base.MySessionContext;

import javax.servlet.http.HttpSession;
import javax.servlet.http.HttpSessionEvent;
import javax.servlet.http.HttpSessionListener;

public class LoginSessionListener implements HttpSessionListener {

	private MySessionContext myc = MySessionContext.getInstance(); 
	//MySessionContext是实现session的读取和删除增加
	// 单例模式

	public void sessionCreated(HttpSessionEvent event) {
		myc.AddSession(event.getSession());
	}

	public void sessionDestroyed(HttpSessionEvent event) {
		HttpSession session = event.getSession();
		myc.DelSession(session);
	}

}

```
然后再web.xml中添加listener
```
  <listener>   
    <listener-class>com.vcooline.wshop.listener.LoginSessionListener</listener-class>   
  </listener>
```
根据sessionId获取Session对象：
```
String sessionId = request.getSession().getId();

HttpSession session = MySessionContext.getSession(sessionId);
```

第二步，在登录过滤器中增加一个方法
```
private Boolean isLogin(HttpServletRequest request){
	 String sessionId = request.getSession().getId();
     HttpSession session = myc.getSession(sessionId);  
     Map<String,HttpSession> userMap = myc.getMap();
     Admin admin = (Admin)request.getSession().getAttribute("USER");
     if(admin == null){
    	 return false;
     }
     if(session != null && userMap != null){
    	 for (String key : userMap.keySet()) {
    		 if(userMap.get(key) != null){
    			 Admin us = (Admin)userMap.get(key).getAttribute("USER");
    			 if(us != null && us.equals(admin) && !sessionId.equals(key)){
    				 userMap.get(key).invalidate();
        			 return true;
    			 }
    		 }
    	 }
     }
	return false;
}
```
	1. 如果当前Session没有用户对象，不做任何处理，直接放行。
	2.如果当前Session包含用户对象，遍历所有Session，找到其它包含当前登录用户信息的Session然后失效它。