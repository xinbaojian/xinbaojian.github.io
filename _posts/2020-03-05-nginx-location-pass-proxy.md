---
layout:     post
title:      "Nginx location proxy_pass 后面的 url 加与不加 / 的区别"
subtitle:   "\"Nginx location proxy_pass 后面的 url 加与不加 / 的区别\"" 
author:     "xin.bj"
header-img: "img/post-bg-2015.jpg"
tags:
    - Spring Boot
---

# Nginx location proxy_pass 后面的 url 加与不加 / 的区别.


这里我们分4种情况讨论

这里我们请求的网站为：**192.168.1.123:80/static/a.html**

**整个配置文件是**

```json
server{
	port  80,
	server name  192.168.1.123

	location /static {
		proxy_pass  192.168.2.321:81
	}

    location /static {
	    proxy_pass  192.168.2.321:81/
    }

    location /static/ {
    	proxy_pass  192.168.2.321:81
    }

    location /static/ {
	    proxy_pass  192.168.2.321:81/
    }
}
```

我们分开来讲：

## 第一种 

1. location后没有 `/`
2. proxy_pass URL后没有 `/`

```json
#192.168.1.123->server name
# :80 ---------> port
#/statc ------->location
#/a.html ------>proxy_pass 

location /static {
	proxy_pass  192.168.2.321:81
}
```

原始网址是：`http://192.168.1.123/static/a.html`

最后网址经过Nginx转向后的实际地址是: `192.168.2.321/static/a.html`

## 第二种

1. location后没有 `/`
2. proxy_pass URL后有 `/`

```json
#192.168.1.123->server name
# :80 ---------> port
#/statc ------->location
#/a.html ------>proxy_pass 

location /static {
	proxy_pass  192.168.2.321:81/
}
```

原始网址是：`http://192.168.1.123/static/a.html`

最后网址经过Nginx转向后的实际地址是: `192.168.2.321/a.html`

## 第三种

1. location后有 `/`
2. proxy_pass URL后没有 `/`

```json
#192.168.1.123->server name
# :80 ---------> port
#/statc ------->location
#/a.html ------>proxy_pass 

location /static/ {
	proxy_pass  192.168.2.321:81
}
```

原始网址是：`http://192.168.1.123/static/a.html`

最后网址经过Nginx转向后的实际地址是: `http://192.168.2.321/static/a.html`

## 第四种

1. location后有 `/`
2. proxy_pass URL后有 `/`

```json
#192.168.1.123->server name
# :80 ---------> port
#/statc ------->location
#/a.html ------>proxy_pass 

location /static/ {
	proxy_pass  192.168.2.321:81/
}
```

原始网址是：`http://192.168.1.123/static/a.html`

最后网址经过Nginx转向后的实际地址是: `http://192.168.2.321/a.html`

## 总结

从这四种我们可以的看出，当nginx里面匹配时可以把端口后的参数分为path1+path2(其中我在上方标注的location属于path1，proxy_pass属于path2)
当 proxy_pass

- 里面是ip:port+/时nginx最后匹配的网址是 proxy_pass的内容加上path2
- 里面是ip:port时nginx最后匹配的网址是 proxy_pass的内容加上path1+path2

