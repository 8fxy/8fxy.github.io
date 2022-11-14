---
layout: post
title: Docker 部署 postgresql
subtitle: 映射主机端口
tags: [Tips]
comments: true
---

# Docker 部署 postgresql

## 1. 安装 Docker

请参考[官方文档](https://docs.docker.com/get-started/)

### 允许非 root 用户管理 Docker
```console
$ sudo add groupadd docker
$ sudo usermod -aG docker $USER
$ newgrp docker 
```

## 2. 部署 Postgresql

### 启动容器 

```console
$ docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```

###  用户名 postgres 首次登陆

进入容器内的终端

```console
$ docker container exec -it some-postgres su postgres
```

修改默认用户 postgres 的密码

```
$ psql
\password postgres
```

输入两次密码并确认

### 添加用户和授权

在 postgres 控制台内，创建新用户 test：

```
CREATE USER test WITH PASSWORD '******';
```

创建数据库，指定拥有者为 dbuser：

```
CREATE DATABASE testdb OWNER test;
```

赋予 dbuser 操作数据库的所有权限：

```
GRANT ALL PRIVILEGES ON DATABASE testdb to test;
```

### 开启登录权限和设置服务状态

确认配置文件位置

```console
$ psql -U postgres -c 'SHOW config_file'
```

修改 `postgresql.conf` 配置文件：

```console
$ sudo vim /etc/postgresql/10/main/postgresql.conf
```

修改或添加下面一行：

```
listen_addresses = '*'
```

修改 `pg_hba.conf` 配置文件：

```console
$ sudo vim /etc/postgresql/10/main/pg_hba.conf
```

修改或添加下面一行：

```text
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             0.0.0.0/0               md5
```

保存文件，重启 postgresql 服务，并设置开机自动启动：

```console
$ sudo systemctl restart postgresql
$ sudo systemctl enable postgresql
```

查看服务状态和关闭服务的命令分别为：

```console
$ sudo systemctl status postgresql
$ sudo systemctl stop postgresql
```


## 3. 将容器端口映射到主机端口

这一步可以让外部用户通过主机端口访问容器内的应用

如果在 docker run 启动的时候没有指定端口映射，可以通过以下步骤添加

```console
$ docker inspect some-container |grep Id  # 查找容器 id
$ docker stop some-container  # 停止容器
$ systemctl stop docker  # 停止 Docker 服务
```

找到容器的配置文件目录

```console
$ cd /var/lib/docker/containers/some-container-id
```

修改 `hostconfig.json` ， 将容器端口映射到主机端口：

```json
"5432/tcp": [{"HostIp":"","HostPort":"5432"}]
```

修改 `config.v2.json`， 在 `ExposedPorts` 中加入要暴露的端口：

```json
ExposedPoerts:{"5432/tcp":{}}
```

重启 Docker 后，外部设备就可以通过 主机IP:5432 访问应用了