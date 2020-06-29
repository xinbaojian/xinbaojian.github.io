---
layout:     post
title:      "JupyterLab安装Java内核"
subtitle:   " \"Jupyter\""
date:       2020-06-29 10:17:13
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - Jupyter
---

# Jupyter 安装Java内核

## Java JDK

- 首先下载最新的[JavaJDK](https://www.oracle.com/java/technologies/javase-downloads.html) jdk>=9
- 配置环境变量
- 测试`java -version`

## 安装jupyter java内核

先查看下jupyter 内核

```shell
jupyter kernelspec list
#删除内核使用
jupyter kernelspec remove java
```

## 安装IJAVA

- 打开github上[IJAVA](https://github.com/SpencerPark/IJava/releases)下载页,下载最新版本，使用unzip解压，解压后的目录结构为：

```
java
install.py
```

- 使用Python安装

  ```python
  python install.py
  ```

> Tips:
>
> 这里我使用这种方式安装出错了，
>
> ```
>  ijava python3 install.py            
> Traceback (most recent call last):
>   File "install.py", line 6, in <module>
>     from jupyter_client.kernelspec import KernelSpecManager
> ModuleNotFoundError: No module named 'jupyter_client'
> ```
>
> 使用以下方式安装成功：
>
> ```shell
> jupyter kernelspec install --user java/
> ```

- 再次查看jupyter内核

  ```shell
  jupyter kernelspec list
  ```

  ```
  Available kernels:
    python3    /home/linuxbrew/.linuxbrew/opt/python@3.8/lib/python3.8/site-packages/ipykernel/resources
    java       /home/xinbj/.local/share/jupyter/kernels/java
  ```

- 重新启动jupyter-lab就可以了