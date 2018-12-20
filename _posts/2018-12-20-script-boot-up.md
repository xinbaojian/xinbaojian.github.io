---
layout:     post
title:      "Ubuntu 18.04 开机启动脚本"
subtitle:   "\"开机启动\"" 
date:       2018-12-20 01:03:15
author:     "xin.bj"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Linux
---

Ubuntu 16.04 开始不再使用initd管理系统，改用systemd.update-rc.d 以及 rc.local 等方法不在生效.通过一些简单的设置，让rc.local 重新可用。


1.建立rc-local.service文件

    sudo vim /etc/systemd/system/rc-local.service

2.将下列内容复制进rc-local.service文件

```
[Unit]
Description=/etc/rc.local Compatibility
ConditionPathExists=/etc/rc.local
 
[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99
 
[Install]
WantedBy=multi-user.target
```

3.创建文件rc.local

    sudo vim /etc/rc.local

4.将下列内容复制进rc.local文件

```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
echo "看到这行字，说明添加自启动脚本成功。" > /usr/local/test.log
# 在 `exit 0` 之前执行想要开机执行的脚本
exit 0
```

5.给rc.local加上权限,启用服务

    sudo chmod +x /etc/rc.local
    sudo systemctl enable rc-local

6.启动服务并检查状态

    sudo systemctl start rc-local.service
    sudo systemctl status rc-local.service

7.重启并检查test.log文件

    cat /usr/local/test.log