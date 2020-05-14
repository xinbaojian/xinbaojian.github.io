---
layout:     post
title:      "搭建Docker Registry"
subtitle:   " \"Docker\""
date:       2020-05-14 09:01:13
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Docker
---



# 搭建Docker Registry

> 服务器IP: 192.168.25.201

## 1.安装Docker

安装最新版Docker,自行百度。

## 2. 运行Registry

```shell
docker run -d -v /data/registry:/var/lib/registry \
	-p 5000:5000 \
	--restart=always \
	--name registry registry:latest
```

参数说明：

-d：后台运行；

-v：把宿主机的/data/registry目录绑定 到 容器/var/lib/registry目录(这个目录是registry容器中存放镜像文件的目录)，来实现数据的持久化；

-p：映射端口；访问宿主机的5000端口就访问到registry容器的服务了；

--restart=always：这是重启的策略，假如这个容器异常退出会自动重启容器；

--name registry：创建容器命名为registry，你可以随便命名；

registry:latest：这个是刚才pull下来的镜像，如果本地无此镜像，会自动下载。

等待启动成功。

## 3. 测试

命令行执行

```shell
curl http://192.168.25.201:5000/v2/_catalog
```

或者浏览器访问

```html
http://192.168.25.201:5000/v2/_catalog
```



# 安装 docker-registry-web

安装WebUI，增加仓库的可读性，方便用户查看。

## 1.拉取镜像

```shell
docker pull hyper/docker-registry-web
```

## 2. 启动WebUI同时链接仓库

```shell
docker run -d --restart=always -p 5001:8080 --name registry-web \
--link registry \
-e REGISTRY_URL=http://192.168.25.201:5000/v2 \
-e REGISTRY_NAME=192.168.25.201:5000 \
--restart=always \
hyper/docker-registry-web
```

**这里的--link 是上面的容器名registry**

## 3.访问Web

http://192.168.25.201:5001

![registry-web](http://cdn.xinbaojian.xyz/image/20200514/AmPSmL399rEk.png?imageslim)



## 4. 客户端修改daemon.json

Linux 端文件在`/etc/docker/daemon.json` 没有就创建

`registry-mirrors` 镜像仓库

`insecure-registries` 仓库私服

要符合`json`格式.不然会报错。

```json
➜  ~ cat /etc/docker/daemon.json 
{
	"registry-mirrors": [
		"http://f1361db2.m.daocloud.io"
	],
	"insecure-registries":[
		"192.168.25.201:5000"
	]
}
```

上传到私服的镜像需要重命名，添加`192.168.25.201:5000`前缀。比如：

```shell
# docker pull busybox
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              f6e427c148a7        36 hours ago        1.15MB
```

为镜像打标签

```shell
docker tag busybox:lastest 192.168.25.201:5000/busybox:v1
```

格式说明：Usage:   docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

192.168.25.201:5000/busybox:v1：这是目标镜像，也是registry私有镜像服务器的IP地址和端口；

上传到镜像服务器

```shell
docker push 192.168.25.201:5000/busybox:v1
```

这是发现报错了

```shell
The push refers to repository [192.168.192.114:5000/busybox]
Get https://192.168.25.201:5000/v2/: http: server gave HTTP response to HTTPS client
```

注意了，这是报错了，需要https的方法才能上传，我们可以修改下仓库端 daemon.json来解决：

```shell
# vim /etc/docker/daemon.json 
{
  "registry-mirrors": [ "https://registry.docker-cn.com"],
  "insecure-registries": [ "192.168.192.114:5000"] # 设置仓库镜像地址
}
```

添加私有镜像服务器的地址，注意书写格式为json，有严格的书写要求，然后重启docker服务：

```shell
systemctl  restart docker
```

