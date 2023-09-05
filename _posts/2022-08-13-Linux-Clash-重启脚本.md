---
layout: post
title: Linux Clash 重启脚本 
subtitle: 下载配置文件和重启服务
tags: [Tips]
comments: true
---

# Linux Clash 重启脚本

新建脚本：

```shell
$ touch ~/bin/reboot_clash.sh
```

写入内容：

```shell
#/usr/bin/zsh

# 下载最新的配置文件
wget -O ~/.config/clash/config.yaml '你的clash订阅地址'
# 下载最新的GeoIP文件
wget -O ~/.config/clash/Country.mmdb https://www.sub-speeder.com/client-download/Country.mmdb

# 杀掉原先启动的clash服务
pkill clash

# 修改默认端口
sed -i 's/7890/6666/' ~/.config/clash/config.yaml
sed -i 's/7891/7777/' ~/.config/clash/config.yaml
sed -i 's/7892/8888/' ~/.config/clash/config.yaml

# 启动clash
# nohup让ssh退出后clash能继续运行
# 将日志输出到.log文件中，用tail -fn 100 clash.log 命令可以查看最新实时100条日志
# 2>&1 将标准错误重定向到标准输出
# & 表示后台运行
nohup clash > ./clash.log 2>&1 &
```

授予脚本可执行权限：

```shell
$ sudo chmod +x ~/bin/reboot_clash.sh
```


创建快捷命令， `vim ~/.bashrc` 添加：

```shell
alias reboot_clash='./bin/reboot_clash.sh'
```

这样在 bash 终端中运行 `reboot_clash` 即可方便地重启Clash

ref. : https://www.cnblogs.com/yuxiayizhengwan/p/16372429.html