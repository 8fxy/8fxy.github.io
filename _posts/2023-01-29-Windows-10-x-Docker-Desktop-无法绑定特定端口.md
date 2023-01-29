---
layout: post
title: Windows 10 x Docker Desktop 无法绑定特定端口
subtitle: 重启Winnat解决
tags: [Docker]
comments: true
---


## 现象描述

项目需要用到一个 pgsql 容器，端口映射 `-p 55000:5432`
Docker Desktop 启动容器后显示端口映射成功
但是无法通过宿主机端口+映射端口号访问，而Docker 内部IP + 内部端口号可以访问。

## 解决方法

手动启动容器发现报错：

> Docker: error response from daemon: Ports are not available: listen tcp 0.0.0.0/50070: bind: An attempt was made to access a socket in a way forbidden by its access permissions.

也就是容器启动了，但端口实际上并没有映射成功，客户端显示的映射成功是假的

搞了半天发现似乎是 `Hyper-V + winnat` 的神秘 bug，cmd 中用以下命令解决：

```cmd
net stop winnat
docker start container_name
net start winnat
```

至于底层原因，参考链接2中看到一个说法：

>有文章说是Hyper-V会随机保留一些端口，具体干什么我也不记得了，问微软去。反正就是会绑定一组随机端口（是一组不是一个），如果你很幸运，保留范围里面有58526，那你不管怎么折腾都不可能连接上WSA。

我...祝微软越办越好！

## 参考链接

1. [Ports are not available: listen tcp 0.0.0.0/50070: bind: An attempt was made to access a socket in a way forbidden by its access permissions](https://stackoverflow.com/questions/65272764/ports-are-not-available-listen-tcp-0-0-0-0-50070-bind-an-attempt-was-made-to)
2. [故事会 好好的程序咋就天天报错呢？WinNAT？](https://www.bilibili.com/read/cv16466587)