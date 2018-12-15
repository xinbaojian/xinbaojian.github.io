---
layout:     post
title:      "在Mac下安装postgresql"
subtitle:   " \"Mac Postgresql\""
date:       2016-12-11 09:00:00
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - Mac
    - Postgresql
---

> postgresql

## 在Mac下安装postgresql

 用brew 安装postgresql

```shell
brew install postgresql
```

然后创建工作目录，并相应的赋予权限

```	
mkdir -p /usr/local/var/postgres
```

初始化数据库文件

```shell
initdb /usr/local/var/postgres/data
```

创建log目录

```
midir -p /usr/local/var/log/postgres
```

指定日志输出并启动。

```shell
pg_ctl start -D /usr/local/var/postgres/data -l /usr/local/var/postgres/postgres.log start
```

停止的话，把后面的start 换成stop 即可.