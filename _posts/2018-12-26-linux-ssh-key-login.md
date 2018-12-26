---
layout:     post
title:      "Linux SSH免密码登录"
subtitle:   "\"SSH免密码登录\"" 
date:       2018-12-26 10:44:15
author:     "xin.bj"
header-img: "img/post-bg-2015.jpg"
tags:
    - Linux
    - SSH
---

设置SSH免密码登录老是记不住文件名称及权限，特此记录一下。

## 第一步：生成密钥

本机和要登录的服务器均需要生成：

Windows 下生成秘钥：

环境准备：

    Git For Windows  - 目的，使用该软件提供的`Git Bash`环境

步骤：

1. 打开Git Bash 终端（在那个目录打开随意，不影响秘钥生成路径）

2. 执行命令 `ssh-keygen -t rsa`;(输入该命令后一路回车就好)。终端出现以下类似画面。

```shell
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):  #回车
Created directory '/home/ubuntu/.ssh'.
Enter passphrase (empty for no passphrase): #回车
Enter same passphrase again: #回车
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:hqbuctGshfIdGyPJ6W0iJFNNh1p8uzJ8zqmi++2x4bA ubuntu@ubuntu
The key's randomart image is:
+---[RSA 2048]----+
|   . .           |
|    = o          |
|   = o .         |
|  o . ..         |
| . o *o.S        |
|o o @oX.         |
| +.++& *         |
|  +**+O          |
|o+EBO+           |
+----[SHA256]-----+
```

这时在 `C:\Users\Administrator\.ssh` 目录下会生成秘钥文件（Administrator 为当前Windows登录用户，替换为你当前使用的用户名）

```shell
➜  .ssh tree
.
├── id_rsa
├── id_rsa.pub
```

Linux环境下生成秘钥：

和上边步骤一样，秘钥生成路径为 `/home/${user}/.ssh`


## 第二步： 设置秘钥

1. 登录服务器，进入 `/home/${user}/.ssh` 目录，生成 `authorized_keys`文件

```shell
touch authorized_keys
```
此时目录结构为：

```
$ tree
.
├── authorized_keys
├── id_rsa
└── id_rsa.pub

```

- authorized_keys:存放远程免密登录的公钥,主要通过这个文件记录多台机器的公钥

- id_rsa : 生成的私钥文件

- id_rsa.pub ： 生成的公钥文件

- know_hosts : 已知的主机公钥清单

- 如果希望ssh公钥生效需满足至少下面两个条件：
    1) .ssh目录的权限必须是700
    2) .ssh/authorized_keys文件权限必须是600

## 第三步：编辑 `authorized_keys` 文件

拷贝本地机器 `.ssh` 目录中 `id_rsa.pub` 文件内容到 `authorized_keys`文件中,例如：

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2Cfd4RGWzXjR5qAhzByqHJMZ9Eh3kJSbiHvG3rWK+DxtFSGj/nw6x7UuoLg/077LsSqLYcNxVoiKKrTDvdEq/5KoB95nU+QMd62dlRlI6Icj2kXXXXXXXXX4Nl0EIB19dD0icU13jOwlRl96ATzqC6Bnq4P4cYC/A9DRJSEiG1wwpFNDOm6nn6U+IT5ZuXY1TjAliWNVTCALdsyN5GKRP7d8YmGBm49FFwXlj0mQYJAWfFrZyuoby61CZt6cfNjINTE9v7VLWrOcNnHu+CXUoCh84jx8clGsgmeGGDnOjHVnDsl4vMqAWUp20kwphP2bSRrli0+z4GnqMfiD/oDF Administrator@635NUY81LL8GTJQ
```

至此，配置结束。在本地打开终端测试 

```shell
#ip替换你自己设置的服务器IP

ssh root@192.168.1.1 
```
可直接免密码登录。


## 使用 `SSH config `配置文件管理SSH链接

在~/.ssh/ 下创建 config文件，并以如下格式编辑配置文件：

```
Host lab
    HostName amazon.com
    User root
    Host 22
    IdentityFile ~/.ssh/id_rsa
```

    - Host： 是我们在输入命令的时候的名字 比如我这里是lab  那么我使用ssh命令的时候需要使用 例如: ssh lab (注意这里是空格)
    - HostName： 是目标主机的主机名，也就是平时我们使用ssh后面跟的地址名称。
    - Port：指定的端口号。
    - User：指定的登陆用户名。
    - IdentifyFile：指定的私钥地址。


`ssh config` 配合 `authorized_keys` 免密登录使用，不要太爽哦~