---
layout: post
title: 文件迁移和服务配置
tags: [PostgreSQL, CentOS]
comments: true
---

### 简介
随着数据库容量的增长，需要将数据库迁移到其它目录，本文将基于CentOS 7和PostgreSQL 10介绍具体操作方法
## 需求
环境准备：
- CentOS 7 系统，以及具有sudo权限的非root用户
- 已正常安装的 Postgresql 10

因涉及到多个目录和文件配置，这里提统一列出：
- 旧数据库目录: /var/lib/pgsql/10/data/
- 系统级配置文件目录: /etc/systemd/system/postgresql-10.service.d
- 新数据库目录(示例): /data/postgresql-10/10/data
## 步骤1: 移动数据库文件
首先确认目前的数据库文件目录
```terminal
$ sudo -u postgres psql
```
在pgsql中查询
```terminal
postgres=# SHOW data_directory;
```
结果
```terminal
	   data_directory       
------------------------------
/var/lib/pgsql/10/data
(1 row)
```
输入 `\q` 断开连接，并关闭数据库服务
```terminal
$ sudo systemctl stop postgresql
$ sudo systemctl status postgresql
```
确认关闭后开始迁移数据库，先确认旧目录的具体结构
```terminal
$ ls /var/lib/pgsql/10/data
```
注意用`rsync -av` 在新旧目录之间同步数据库文件，`-a` 表示完整保留文件权限设置，`-v` 便于观察进度
>**注意**：确保 rsync 命令中新目录路径尾部没有反斜杠 `/`，特别是按tab使用自动补全可能会添加反斜杠。有无反斜杠的具体的区别详见参考3

创建新目录
```terminal
$ mkdir /data/postgresql-10
```
有时候一台服务器上会有多种版本的postgresql，可以只移动指定版本对应的路径，也可以将原目录整体迁移，我这里选择整体迁移
```
$ sudo rsync -av /var/lib/postgresql /data/postgresql-10
```
复制完成后，先不要急着删除或移动旧的数据库文件，以防数据库不能正常启动
## 步骤2: 指定数据库新路径
修改新目录中的配置文件
```
$ vim /data/postgresql-10/10/data/postgresql.conf
```
修改 `data_directory` 的值为新数据目录：
```text
data_directory = '/data/postgresql-10/10/data'
```
然后要修改系统中数据库服务的配置，你可以直接修改原始配置文件，但由于数据库升级时这个文件会可能会被覆盖导致问题，所以这里推荐用新建覆写配置文件的方法
先新建一个 `.conf` 后缀的文件，例如
```terminal
$ cd /etc/systemd/system/postgresql-10.service.d
$ mkdir override.conf
```
编辑 `override.conf` 新增以下内容
```text
[Service]
Environment=PGDATA=/data/postgresql-10/10/data
```
注意，这里修改的是操作系统级别的服务配置，而非数据库本身的配置文件
## 步骤3: 重启数据库服务
重新载入配置
```ternimal
$ systemctl daemon-reload
```
重启数据库，检查状态
```terminal
$ systemctl restart postgresql-10
$ sudo systemctl status postgresql
```
确认数据库服务完全正常后，再处理旧目录中的文件

##### 参考
1. [How To Move a PostgreSQL Data Directory to a New Location on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-move-a-postgresql-data-directory-to-a-new-location-on-ubuntu-16-04)
2. [CentOS7系统下PostgreSQL数据库数据目录指定与迁移](https://blog.csdn.net/ScienceRui/article/details/119543088)
3. [rsync命令中的路径反斜杠](https://blog.51cto.com/u_79854/393475#:~:text=%E7%AE%80%E5%8D%95%E6%80%BB%E7%BB%93%E4%B8%80%E4%B8%8B%EF%BC%9A%E6%BA%90%E7%9B%AE%E5%BD%95%E5%B0%BE%E9%83%A8%E7%9A%84%E8%B7%AF%E5%BE%84%E5%8F%8D%E6%96%9C%E6%9D%A0%E6%98%AF%E5%91%8A%E8%AF%89%20rsync%20%E5%A4%8D%E5%88%B6%E8%BF%99%E4%B8%AA%E7%9B%AE%E5%BD%95%E9%87%8C%E7%9A%84%E6%89%80%E6%9C%89%E5%86%85%E5%AE%B9%E5%88%B0%E7%9B%AE%E6%A0%87%E7%9B%AE%E5%BD%95,%EF%BC%8C%E5%A6%82%E6%9E%9C%E4%B8%8D%E5%8A%A0%E7%9B%AE%E5%BD%95%E5%B0%BE%E9%83%A8%E8%B7%AF%E5%BE%84%E5%8F%8D%E6%96%9C%E6%9D%A0%EF%BC%8C%E9%82%A3%E4%B9%88%E8%BF%99%E4%B8%AA%E7%9B%AE%E5%BD%95%E6%9C%AC%E8%BA%AB%E4%BC%9A%E8%A2%AB%E5%A4%8D%E5%88%B6%E5%88%B0%E7%9B%AE%E6%A0%87%E7%9B%AE%E5%BD%95%E4%B8%AD%E3%80%82%20%E6%88%91%E4%BB%AC%E4%B8%80%E8%88%AC%E9%83%BD%E6%98%AF%E6%83%B3%E8%A6%81%E6%8A%8A%E6%BA%90%E7%9B%AE%E5%BD%95%E7%9A%84%E5%86%85%E5%AE%B9%E5%90%8C%E6%AD%A5%E5%88%B0%E7%9B%AE%E6%A0%87%E7%9B%AE%E5%BD%95%E4%B8%AD%EF%BC%8C%E6%89%80%E4%BB%A5%E5%A6%82%E6%9E%9C%E4%BD%A0%E5%AE%9E%E5%9C%A8%E8%AE%B0%E4%B8%8D%E6%B8%85%E8%BF%99%E4%B8%AA%E5%8C%BA%E5%88%AB%EF%BC%8C%E9%82%A3%E4%B9%88%E5%9C%A8%E6%BA%90%E7%9B%AE%E5%BD%95%E7%9A%84%E6%9C%80%E5%90%8E%E5%8A%A0%E4%B8%8A%20%E2%80%9C%2F%2A%E2%80%9D%20%E6%98%AF%E6%9C%80%E4%BF%9D%E9%99%A9%E7%9A%84%E6%96%B9%E6%B3%95%E3%80%82)