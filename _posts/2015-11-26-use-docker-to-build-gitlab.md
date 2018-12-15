---
layout:     post
title:      "使用Docker镜像搭建Gitlab服务"
subtitle:   " \"Docker\""
date:       2015-11-26 10:57:26
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Docker
---

> Docker搭建Gitlab

docker用来隔离应用还是很方便的，一来本身的操作较为简单，二来资源占用也比虚拟机要小得多，三来也较为安全，因为像数据库这样的应用不会再全局暴露端口，同时应用间的通信通过加密和端口转发，更加安全。

Gitlab是目前比较流行的开源类Github代码管理平台。Gitlab使用Rails开发，使用PostgreSQL或MySQL数据库，Redis做缓存。一般自己搭建私有代码仓库，Gitlab通常是首选。这里简单介绍一下dockerized Gitlab。

Gitlab的docker镜像早已有人做好了，并且维护相当不错。大家可以前往其GitHub仓库了解该镜像的情况。官方repo的readme中已经有详细的安装配置方案，这里我简单的梳理一下部署流程。

<!--more-->

### 安装Docker

可以使用国内公司DaoCloud提供的服务，Docker官方安装速度很慢。
参考地址：[Docker 极速下载](http://get.daocloud.io/)

###下载docker-gitlab

#### 下载docker-gitlab最新版,
镜像推荐使用`sameersbn/gitlab`

```
docker pull sameersbn/gitlab:latest
```
如果你使用了DaoCloud的加速服务，则可以这样下载
```
dao pull sameersbn/gitlab:latest
```

### 安装PostgreSQL

Gitlab推荐使用PostgreSQL作为数据库。既然使用了docker，那么我们为何不考虑把所有的组件都用docker包装起来？我们一样可以下载PostgreSQL的镜像完成安装，这种安装更加便捷。

#### 下载PostgreSQL镜像：

```
docker pull sameersbn/postgresql:latest
```

如果你使用了DaoCloud的加速服务，则可以这样下载

```
dao pull sameersbn/postgresql:latest
```

然后我们要为数据库默认的表空间建立目录以存放数据：

```
mkdir -p /opt/gitlab/postgresql
```

这里`/opt/gitlab/postgresql`部分可以替换成你自己希望建立的地址。

最后使用以下命令行启动数据库：
```
docker run --name=gitlab-postgresql -d \
  -e 'DB_NAME=gitlabhq_production' -e 'DB_USER=gitlab' -e 'DB_PASS=password' \
  -v /opt/gitlab/postgresql:/var/lib/postgresql \
  sameersbn/postgresql:latest
```
> 这里，"-e"选项后面的内容请不要随意变更，这里的配置都是Gitlab默认的数据库配置，如果没有在后面Gitlab镜像启动的设置里面做相应的修改的话，这里的修改会让程序无法正常运行。

### 安装Redis

同样，我们可以使用docker来安装Redis：

```
docker pull sameersbn/redis:latest
or
dao pull sameersbn/redis:latest
```
redis映射目录
```
mkdir -p /opt/gitlab/redis
```

然后启动它:

```
docker run --name gitlab-redis -d \
    --volume /opt/gitlab/redis:/var/lib/redis \
    sameersbn/redis:latest
```

### 启动gitlab

在最终启动Gitlab之前，我们还需要为Gitlab创建一个目录用来存放提交上来的代码，docker-gitlab内部使用`/srv/docker/gitlab/gitlab`这个目录存放代码，我们在容器外部创建一个目录然后在启动的时候挂载到这个路径即可：

```
mkdir -p /opt/gitlab/data
mkdir -p /opt/gitlab/backups
```
在完成上面所有的步骤以后，我们可以用以下命令启动Gitlab：

```
docker run --name gitlab -d \
    --link postgresql:postgresql --link redis:redisio \
    --publish 10022:22 --publish 10080:80 \
    --env 'GITLAB_PORT=10080' --env 'GITLAB_SSH_PORT=10022' \
    --env 'GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alpha-numeric-string' \
	-e 'GITLAB_HOST=192.168.1.166' \
	-e 'SMTP_ENABLED=true' \
	-e 'SMTP_DOMAIN=www.126.com' \
	-e 'SMTP_HOST=smtp.126.com' \
	-e 'SMTP_PORT=25' \
	-e 'SMTP_STARTTLS=false' \
	-e 'SMTP_USER=you_email' \
	-e 'SMTP_PASS=you_email_password' \
    --volume /opt/gitlab/data:/home/git/data \
    sameersbn/gitlab:latest
```
> SMTP* 环境变量默认使用Gmail服务，你可以替换为你常用的邮箱，比如上边使用126邮箱
> you_email,you_email_password 替换为你的邮箱和密码

上面的命令将使用10080作为Gitlab的Web访问端口，10022将作为ssh push和pull代码的端口。
在本地可以使用浏览器打开http://localhost:10080 
来访问Gitlab，初始登录网站使用root账户，用户名为`root`，密码为：`5iveL!fe`，登录后需要立即修改密码。

这里解释一下各参数：

* -d: 后台运行
* -e：配置Gitlab运行的环境变量，这个参数很重要，具体有哪些环境变量，后面列举
* -p: 端口转发规则
* -v: 共享目录挂载，即docker容器内外数据共享

Gitlab的环境变量配置比较多，这里列举一下比较重要的Gitlab的环境变量：

* GITLAB_HOST: 这个是Gitlab服务器的hostname，你需要将此设定为网站的域名或者ip（不带端口号），默认值为localhost，这个值会被Gitlab用来生成repo的链接，所以必须要设置。否则，在创建的repo中，会发现所有的repo链接都是以localhost为hostname。
* GITLAB_PORT Gitlab网站的访问端口，这里的设置要结合端口转发一起设置，否则会导致网站无法访问，默认值为80
* GITLAB_SSH_PORT Gitlab的SSH代码提交方式使用的SSH端口，这里的设置要结合端口转发一起设置，否则会导致代码无法提交，默认值为22。如果是在VPS上部署，这个值请使用别的端口，比如上面提到的10022端口，否则会与VPS原本的SSH端口产生冲突，造成SSH无法登录VPS
* GITLAB_BACKUPS Gitlab的自动备份配置，有disable, daily, weekly, monthly四个可选值，默认为disable。建议打开自动备份
* GITLAB_BACKUP_DIR Gitlab自动备份目录，默认值为/home/git/data/backups

详细参数设置请参考`参考资料1`






参考资料：
> 1[sameersbn/gitlab镜像文档](https://hub.docker.com/r/sameersbn/gitlab/#available-configuration-parameters)
> 2[docker-gitlab部署](http://segmentfault.com/a/1190000002421271) 
> 3[GitLab搭建与维护](http://www.tuicool.com/articles/bYbi2mJ)