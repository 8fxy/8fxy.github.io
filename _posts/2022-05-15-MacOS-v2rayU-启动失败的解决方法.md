---
layout: post
title: MacOS v2rayU启动失败的解决方法
subtitle: 设置订阅后无法启动
tags: [Tips]
comments: true
---

- 环境：MacOS
- 现象：设置订阅后，程序闪退，无法正常启动
- 解决方法：删除以下两个配置文件，重新启动

```zsh

rm -rf ~/Library/LaunchAgents/yanue.v2rayu.v2ray-core.plist

rm -rf ~/Library/Preferences/net.yanue.V2rayU.plist

```
