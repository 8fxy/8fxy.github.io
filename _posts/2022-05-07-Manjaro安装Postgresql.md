---
layout: post
title: Manjaro安装Postgresql
subtitle: 踩坑记录
tags: Manjaro
comments: true
---

# 1. 环境
```shell
$ cat /etc/lsb-release
DISTRIB_ID=ManjaroLinux
DISTRIB_RELEASE=21.2.6
DISTRIB_CODENAME=Qonos
DISTRIB_DESCRIPTION="Manjaro Linux"

$ psql --version
psql (PostgreSQL) 14.2
```

# 2. 安装
- 这一步遇到一个恶心的坑：因为服务器很久没更新了，数据库版本比较新，两个内核的包没有更新（icu和gblic），导致无法运行initdb
- 解决：先更新pacman和yay
```shell
$ sudo pacman -Syy
$ yay
$ yay postgres postgis
```

# 3. 初始化和服务
```shell
$ sudo su postgres -l 
$ initdb --locale $LANG -E UTF8 -D '/var/lib/postgres/data/'
$ exit
```

**配置项含义：**  
- -locale is the one defined in /etc/locale.conf.  
- -E is the default encoding for new databases.  
- -D is the default location for storing the database cluster.  

**查看服务状态：**
```shell
$ sudo systemctl status postgresql
```

**重启服务：**
```shell
$ sudo systemctl restart postgresql
```

**设置开机启动：**
```shell
$ sudo systemctl enable --now postgresql.service
```

# 4. 首次登陆
安装PostgreSQL数据库服务器后，默认情况下，它将创建一个用户`postgres`，其角色为`postgres`。  
它还会创建一个具有相同名称`postgres`的Linux系统帐户。  
因此，要连接到Postgres服务器，请以Postgres用户身份登录到您的系统并连接数据库.  
```shell
$ sudo su - postgres
$ psql
```
然后就可以使用sql query了


# 参考
1. [如何在Manjaro 20上安装PostgreSQL](https://www.yundongfang.com/Yun40753.html)
2. [Install PostgreSQL on Manjaro and set it up for Django](https://gist.github.com/marcorichetta/af0201a74f8185626c0223836cd79cfa)

