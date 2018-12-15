---
layout:     post
title:      "自搭Ngrok实现内网穿透"
subtitle:   " \"Ngrok\""
date:       2016-12-09 16:47:57
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - Ngrok
---

> 内网穿透

#### 一、准备工作

搭建ngrok服务需要在**公网有一台vps**，需要一个自己的**域名**。已有域名的，可以建立一个<u>子域名</u>，用于关联ngrok服务，这样也不会干扰原先域名提供的服务。(不用域名的方式也许可以，但我没有试验过。）



安装必要的工具

```shell
sudo apt-get install build-essential git mercurial
```



#### 二、实操步骤

我的VPS安装的是 **Ubuntu 16.04.1 LTS**，并安装了 Golang 1.7(**go version go1.7.1 linux/amd64**)。Golang是编译ngrokd和ngrok所必须的，建议直接从golang官方下载对应平台的二进制安装包（国内可以从 golangtc.com上下载，速度慢些罢了）。



##### 1、安装Golang

下载Golang安装包，[官网](https://golang.org/dl/),[Golangtc](http://golangtc.com/download) ,建议选择最新下载，然后解压到 `/usr/local`,例如：

```shell
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
```

配置环境变量：

在~/.bashrc 或者 ~/.profile 或者 /etc/profile 中配置如下变量

```shell
#set golang environment
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/you/go/workspace/path
export PATH=$PATH:$GOPATH:$GOPATH/bin
```

执行`source ~/.bashrc`  or  `source ~/.profile`   or  `source /etc/profile`在当前终端重新加载环境变量或者新开一个终端。

执行`go version` 出现如 `go version go1.7.1 linux/amd64` 则安转成功。(各系统版本信息不同)

##### 2、下载Ngrok源码

```shell
cd ~
git clone https://github.com/inconshreveable/ngrok.git
export export GOPATH=~/ngrok
# 这里编译Ngrok时需要以Ngrok项目为 GOPATH
```

##### 3、 生成自签名证书

使用ngrok.com官方服务时，我们使用的是官方的SSL证书。自建ngrokd服务，我们需要生成自己的证书，并提供携带该证书的ngrok客户端。

证书生成过程需要一个**NGROK_BASE_DOMAIN**。 以ngrok官方随机生成的地址693c358d.ngrok.com为例，其NGROK_BASE_DOMAIN就是"ngrok.com"，如果你要 提供服务的地址为"example.tunnel.xinbaojian.me"，那NGROK_BASE_DOMAIN就应该 是"tunnel.xinbaojian.me"。

Tips:

​	NGROK_BASE_DOMAIN:*顶级域名则为:xinbaojian.me*

这里我们以NGROK_BASE_DOMAIN为顶级域名为例，生成证书命令如下：

```shell
openssl genrsa -out base.key 2048
openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=xinbaojian.me" -out base.pem
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=xinbaojian.me" -out server.csr
openssl x509 -req -in server.csr -CA base.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt	
```

执行玩上述命令，在ngrok目录就会生成6个文件

```shell
-rw-rw-r--  1 baojian baojian 1679 12月  7 12:13 rootCA.key
-rw-rw-r--  1 baojian baojian 1123 12月  7 12:13 rootCA.pem
-rw-rw-r--  1 baojian baojian   17 12月  7 12:13 rootCA.srl
-rw-rw-r--  1 baojian baojian 1005 12月  7 12:13 server.crt
-rw-rw-r--  1 baojian baojian  903 12月  7 12:13 server.csr
-rw-rw-r--  1 baojian baojian 1675 12月  7 12:13 server.key
```

ngrok通过bindata将ngrok源码目录下的assets目录（资源文件）打包到可执行文件(ngrokd和ngrok)中 去，assets/client/tls和assets/server/tls下分别存放着用于ngrok和ngrokd的默认证书文件，我们需要将它们替换成我们自己生成的：(**因此这一步务必放在编译可执行文件之前**)

**拷贝证书**

```shell
cp base.pem assets/client/tls/ngrokroot.crt
cp server.crt assets/server/tls/snakeoil.crt
cp server.key assets/server/tls/snakeoil.key
```

##### 3、编译ngrokd 和 ngrok

由于众所周知的原因，国内服务器编译前需要替换log4go的地址

```shell
# 替换下载源地址
sed -i 's#code.google.com/p/log4go#github.com/keepeye/log4go#' ~/ngrok/src/ngrok/log/logger.go
```

在`ngrok`目录下执行如下命令，编译服务端 ngrokd:

```shell
make release-server
#make server 为编译debug版本	

bin/go-bindata -nomemcopy -pkg=assets -tags=release \
        -debug=false \
        -o=src/ngrok/client/assets/assets_release.go \
        assets/client/…
bin/go-bindata -nomemcopy -pkg=assets -tags=release \
        -debug=false \
        -o=src/ngrok/server/assets/assets_release.go \
        assets/server/…
go get -tags 'release' -d -v ngrok/…
code.google.com/p/log4go (download)
go install -tags 'release' ngrok/main/ngrokd
```

编译成功，编译文件为 `~/ngrok/bin/ngrokd`,这就是服务端程序。

在`ngrok`目录下执行如下命令，编译客户端 ngrokd:

编译客户端之前，先确定客户端执行环境，并更改环境变量，进行跨平台编译，如我的树莓派Ubuntu Mate：

```shell
export GOOS=linux
export GOARCH=arm
```

| 系统(GOOS) | 架构(GOARCH)32位 | 架构(GOARCH)64位 |
| :------: | :-----------: | :-----------: |
|  linux   |     amd64     |      386      |
| windows  |     amd64     |      386      |
|  darwin  |     amd64     |      386      |
|  linux   |      arm      |      arm      |

```shell
make release-client
#make client 为编译debug版本

bin/go-bindata -nomemcopy -pkg=assets -tags=release \
        -debug=false \
        -o=src/ngrok/client/assets/assets_release.go \
        assets/client/…
bin/go-bindata -nomemcopy -pkg=assets -tags=release \
        -debug=false \
        -o=src/ngrok/server/assets/assets_release.go \
        assets/server/…
go get -tags 'release' -d -v ngrok/…
go install -tags 'release' ngrok/main/ngrok
```

编译成功，编译文件为 `~/ngrok/bin/linux_arm/ngrok`,这就是服务端程序。(平台名字的文件夹下)

注意：GOPATH务必为ngrok目录。如：`export GOPATH=~/ngrok`,可以执行 `go env查看GOPATH路径`

##### 4、设置域名

域名添加两条A记录

顶级域名

```
* 0.0.0.0
@ 0.0.0.0
www 0.0.0.0
```

二级域名

```
*.ngrok 0.0.0.0
ngrok 0.0.0.0
```



#### 三、启动服务

##### 启动服务端

```shell
./ngrokd -domain="xinbaojian.me" -httpAddr=":8080" -httpsAddr=":8081"
```

##### 启动客户端

```shell
./ngrok -subdomain testing -config=ngrok.cfg 80
# 默认使用http/https 协议
./ngrok -subdomain testing -config=ngrok.cfg -proto tcp 80
# 使用tcp协议
```

设置开机启动项

```shell
# 编辑rc.load
sudo vim /etc/rc.load
nohup /home/baojian/client-ngrok/ngrok -config=/home/baojian/.ngrok -log=stdout start ssh git > /home/baojian/client-ngrok/log/ngrok.log 2>&1 &
# 路径要用绝对路径
```



#### 四、 配置nginx代理

最后一个问题，就是微信公共平台等只支持80端口的，但是我vps的80端口已经被nginx用了，所以ngrok无法再使用80端口，怎么办呢？既然已经有了nginx，那么直接反向代理走起~~，创建一个新的配置文件，输入:

```json
server {
        server_name     ~^(?<subdomain>\w+)\.ngrok\.lylinux\.org$;
        listen 80;
        keepalive_timeout 70;
        proxy_set_header "Host" $host:8081;
        location / {
                proxy_pass_header Server;
                proxy_redirect off;
                proxy_pass http://127.0.0.1:8081;
        }
        access_log off;
        log_not_found off;
}
```

端口号要和上面配置的端口号一致，这样不用输端口号就可以直接使用代理了～～。

#### 五、映射给局域网其它机器

```shell
./ngrok -config ngrok.cfg -subdomain 1 192.168.2.53:3000
```





启动服务的高级用法：参见参考连接 [「翻译」ngrok 1.X 配置文档](https://imlonghao.com/28.html)



参考连接：

[Run Ngrok on Your Own Server Using Self-Signed SSL Certificate](https://www.svenbit.com/2014/09/run-ngrok-on-your-own-server/)

[搭建自己的ngrok服务](http://tonybai.com/2015/03/14/selfhost-ngrok-service/)

[翻译」ngrok 1.X 配置文档](https://imlonghao.com/28.html)

[ubuntu安装ngrok并使用nginx代理](https://www.lylinux.org/ubuntu%E5%AE%89%E8%A3%85ngrok%E5%B9%B6%E4%BD%BF%E7%94%A8nginx%E4%BB%A3%E7%90%86.html)