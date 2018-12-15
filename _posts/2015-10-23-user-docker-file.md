---
layout:     post
title:      "使用Dockerfile创建镜像"
subtitle:   " \"Docker\""
date:       2015-10-23 15:25:12
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Docker
---

> 创建镜像

如何使用Dockerfile创建镜像？

Dockerfile包含创建镜像所需要的全部指令。基于在Dockerfile中的指令，我们可以使用`Docker build`命令来创建镜像。通过减少镜像和容器的创建过程来简化部署。

Dockerfile支持支持的语法命令如下：

	INSTRUCTION argument

指令不区分大小写。但是，命名约定为全部大写。

Tips:
	
	INSTRUCTION argument,不要问我这是什么，我也不知道，看了官网定义，
	大概理解为 `Dockerfile命令 参数`  这样的格式吧。。

<!--more-->

```
FROM ubuntu:14.04

## 设置作者信息
MAINTAINER	http://xinbaojian.xyz xinbaojian <xinbaojian@126.com>

# 使用阿里云镜像

# 备份原源列表
RUN mv /etc/apt/sources.list /etc/apt/sources.list.backup
# 替换阿里云源
ADD ./sources.list /etc/apt/sources.list

# 更新镜像源

RUN apt-get update

RUN apt-get upgrade -y

```

现在创建一个目录，并且创建一个 Dockerfile

	mkdir baojian
	cd baojian
	touch Dockerfile

把上面的内容填充在Dockerfile中，sources.list 可以从这里下载 [sources.list](https://github.com/widuu/Dockerfile/blob/master/sources.list)

如果不想下载，可以进行如下操作

	cd baojian
	touch sources.list
	#拷贝下边sources.list 的内容
	vi sources.list
	#按 i 进入编辑模式
	ctrl + v 粘贴
	:wq    # 保存并退出vi 编辑器


sources.list 内容
```
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
```

baojian/ 的目录结构如下
```
.
└── baojian
    ├── Dockerfile
    └── sources.list
```

在baojian 目录下执行如下命令

	docker build -t ubuntu:baojian .

我们使用 docker build 命令并指定 -t 标识(flag)来标示属于 ubuntu ，镜像名称为 ubuntu,标签是 baojian


我们可以使用新的镜像来创建容器：

	docker run -t -i ubuntu:baojian /bin/bash
	root@8196968dac35:/#

> 注意：这里只是简单的介绍一下如何创建镜像。我们跳过了很多你可以使用的其它指令。

这就是一个以镜像 ubuntu:baojian 为基础镜像的 容器了，你可以在里面安装你想要的工具

安装完成后你也可以使用如下命令，把它们提交进基础镜像里

	exit 退出容器
	docker ps -l 查看刚才镜像的 CONTAINER ID
	docker commit -m="added some software" -a="baojian" 7170ceab520d ubuntu:baojian
	output:add5a99769283525e33e84d5158b82995d9a11d475a57d09727f06df2a1e71f8

这里我们使用了 `docker commit` 命令。这里我们指定了两个标识(flags) `-m` 和 `-a` 。`-m` 标识我们指定提交的信息，就像你提交一个版本控制。`-a` 标识允许对我们的更新来指定一个作者。

我们也指定了想要创建的新镜像容器来源 (我们先前记录的ID) `7170ceab520d` 和 我们指定要创建的目标镜像：

`ubuntu:baojian`

以后再以 `ubuntu:baojian` 为基础镜像生成的 容器 ，就包含上边安装过的软件了。

参考资料：
> 1) [Docker入门教程（三）Dockerfile](http://dockone.io/article/103)
> 2) [使用 Docker 镜像](http://docker.widuu.com/userguide/dockerimages.html)
> 3) [widuu/Dockerfile](https://github.com/widuu/Dockerfile)
> 4) [Docker 2 -- 关于Dockerfile](http://blog.tankywoo.com/docker/2014/05/08/docker-2-dockerfile.html)