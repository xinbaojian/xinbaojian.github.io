---
layout:     post
title:      "Ubuntu 16.04 LTS 下使用Docker安装Gitlab"
subtitle:   "\"Docker 安装 Gitlab\"" 
author:     "xin.bj"
header-img: "img/post-bg-2015.jpg"
tags:
    - Linux
    - Ubuntu
    - Docker
    - Gitlab
---

官方文档地址： [Gitlab官方文档](https://docs.gitlab.com/omnibus/docker/)


## 预配置Docker容器
您可以通过将环境变量添加GITLAB_OMNIBUS_CONFIG到docker run命令来预配置GitLab Docker映像。此变量可以包含任何gitlab.rb设置，并在加载容器gitlab.rb文件之前进行评估。这样，您可以轻松配置GitLab的外部URL，从Omnibus GitLab模板进行任何数据库配置或任何其他选项 。

注意：包含的设置GITLAB_OMNIBUS_CONFIG不会写入gitlab.rb配置文件，而是在加载时进行评估。

这是一个设置外部URL并在启动容器时启用LFS的示例：

```sh
docker run --detach \
    --hostname gitlab.qi-tech.com \
    --env GITLAB_OMNIBUS_CONFIG="external_url 'http://192.168.25.94:10080/'; gitlab_rails['lfs_enabled'] = true; gitlab_rails['gitlab_shell_ssh_port'] = 10022" \
    -p 10022:22 -p 10080:10080 \
    --name gitlab \
    --restart always \
    --volume /data/gitlab/config:/etc/gitlab \
    --volume /data/gitlab/logs:/var/log/gitlab \
    --volume /data/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```

启动成功后访问 [http://192.168.25.194:10080](http://192.168.25.194:10080) (ip 替换为您自己的IP)

这时，需要默认账户(root) 的密码，设置完登录即可。

Gitlab 已默认支持国际化，在 `Profile`中设置显示语言即可。

### 配置 `/data/gitlab/config/gitlab.rb`

#### 配置邮件服务

以腾讯企业邮箱为例:

```ruby
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "gitlab@fayfox.com"
gitlab_rails['smtp_password'] = "******"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['gitlab_email_from'] = 'gitlab@fayfox.com'
```

配置完后重启 gitlab 

    docker restart gitlab

如果需要测试邮件配置是否正确，可以进入容器测试

进入容器

    docker exec -it gitlab /bin/bash

进入 console

    gitlab-rails console

输入以下命令测试(邮箱改为您自己的邮箱)：

    Notify.test_email('gitlab@126.com', 'Message Subject', 'Message Body').deliver_now

收到邮件即配置成功了。


#### 修改gitlab 时区

gitlab 时区默认是 UTC时区

    vim /data/gitlab/config/gitlab.rb

    gitlab_rails['time_zone'] = 'Asia/Shanghai'

配置完后重启 gitlab 