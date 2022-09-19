---
layout: post
title: Manjaro Linux 远程 Windows 
subtitle: 基于Xrdp
tags: [Tips]
comments: true
---


# Manjaro Linux 远程 Windows

## 准备
- 确保 Windows 可以接受远程连接
- Manjaro Linux 安装rdesktop
```shell
$ yay -S rdesktop
```

- （可选）安装vnc

```shell
$ yay -S tigervnc-server
```

>如果远程到 windows 系统则不需要安装vnc

## 安装xrdp

```shell
$ yay -S xrdp
$ yay -S xorgxrdp-git
```

## 启动服务和修改配置

```shell
$ sudo systemctl enable xrdp.service
$ sudo systemctl xrdp-sesman.service
$ edit ~/.xinitrc
```

注释以下行：

```text
#exec $(get_session "$1")
```

增加以下行：

```text
exec dbus-launch --sh-syntax startplasma-x11
```

reboot 之后，可以通过 Manjaro Linux 远程 Windows 10：

```shell
$ rdesktop -u YourUserName -p YourPassword WindowsIPAddress:3389
```