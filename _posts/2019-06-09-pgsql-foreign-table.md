---
layout:     post
title:      "PgSql 创建外部表"
subtitle:   "\"PgSql 创建外部表\"" 
author:     "xin.bj"
header-img: "img/post-bg-2015.jpg"
tags:
    - PgSql
---

## postgres_fdw

`postgres_fdw`模块提供了外部数据封装器 `postgres_fdw`， 它可以被用来访问存储在外部PostgreSQL服务器中的数据。

这个模块提供的功能基本上覆盖了较老的dblink模块的功能。 但是postgres_fdw提供了更透明且更兼容标准的语法来访问远程表， 并且可以在很多情况下给出更好的性能。

要使用postgres_fdw来为远程访问做准备：

    1. 使用CREATE EXTENSION来安装postgres_fdw扩展。

    2. 使用CREATE SERVER创建外部服务器对象， 以表示你想连接的每一个远程数据库。指定除了user和 password之外的连接信息作为该服务器对象的选项。

    3. 使用CREATE USER MAPPING， 为每个要允许访问外部服务器的数据库用户创建用户映射。 指定远程用户名和口令作为用户映射的user和password选项。

    4. 使用CREATE FOREIGN TABLE或IMPORT FOREIGN SCHEMA， 为每一个你想访问的远程表创建一个外部表。外部表的列必须匹配被引用的远程表。 但是，如果指定正确的远程名称作为外部表对象的选项，则可以使用与远程表不同的表和/


现在你只需要从一个外部表SELECT来访问存储在它的底层的远程表中的数据。 你也可以使用INSERT、UPDATE或DELETE修改远程表 （当然，你在你的用户映射中已经指定的远程用户必须具有做这些事情的权限）。

请注意，postgres_fdw目前不支持带有ON CONFLICT DO UPDATE 子句的INSERT语句。但是，如果省略了唯一索引推理规范， 则支持ON CONFLICT DO NOTHING子句。

我们通常推荐一个外部表的列被声明为与被引用的远程表列完全相同的数据类型和排序规则 （如果适用）。尽管postgres_fdw目前已经能够容忍在需要时执行数据类型转换， 但是由于远程服务器解释WHERE子句时可能会与本地服务器有所不同， 所以当类型或排序规则不匹配时可能会发生奇怪的语义异常。

注意一个外部表可以声明比底层的远程表较少的列，或者使用一种不同的列序。 与远程表列的匹配是通过名字而不是位置进行的。


## 使用例子

### 1. 安装扩展

这里是一个用postgres_fdw创建外部表的例子。首先安装该扩展：

```shell
CREATE EXTENSION postgres_fdw;
```

### 2. 创建外部服务器

然后使用`CREATE SERVER`创建一个外部服务器。在这个例子中 我们希望连接到一个位于主机`127.0.0.1`上 并且监听5432端口的PostgreSQL服务器。 在该远程服务器上要连接的数据库名为`foreign_db`：

```shell
CREATE SERVER foreign_server
        FOREIGN DATA WRAPPER postgres_fdw
        OPTIONS (host '127.0.0.1', port '5432', dbname 'foreign_db');
```

### 3. 定义用户映射

也需要一个用`CREATE USER MAPPING` 定义的用户映射来标识将在远程服务器上使用的角色：

```shell
CREATE USER MAPPING FOR local_user
        SERVER foreign_server
        OPTIONS (user 'foreign_user', password 'password');
```

### 4. 创建外部表

```shell
CREATE FOREIGN TABLE foreign_table (
        id integer NOT NULL,
        data text
)
        SERVER foreign_server
        OPTIONS (schema_name 'public', table_name 'foreign_table');
```

### 5. 导入外部表

从服务器`foreign_server`上的远程模式 `public` 导入表定义，在本地模式 films 中创建外表：

```shell
IMPORT FOREIGN SCHEMA public FROM SERVER foreign_server INTO public;
```

如上所述，但只导入两个表actors和directors（如果存在）

```shell
IMPORT FOREIGN SCHEMA public LIMIT TO (foreign_table, foreign_table_b) FROM SERVER foreign_server INTO public;
```

### 删除各项定义

```shell
//删除外部表
drop foreign table foreign_table;
//删除mapping 关系
drop user mapping for postgres server foreign_server;
//删除server
drop server foreign_server;
//删除扩展
drop extension postgres_fdw;
```


