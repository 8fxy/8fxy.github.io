## 安装 Docker

## 允许非 root 用户管理 Docker
```console
$ sudo add groupadd docker
$ sudo usermod -aG docker $USER
$ newgrp docker 
```

## 部署 Postgresql
### start a postgres instance

```console
$ docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```

###  用户名 postgres 首次登陆

```console
$ docker container exec -it some-postgres su postgres
$ psql
$ \password postgres
```

输入两次密码并确认

### 用户添加和授权

在控制台内，创建新用户test：

```
CREATE USER test WITH PASSWORD '******';
```

创建数据库，指定拥有者为dbuser：

```
CREATE DATABASE testdb OWNER test;
```

赋予dbuser操作数据库的权限：

```
GRANT ALL PRIVILEGES ON DATABASE testdb to test;
```

### 开启登录权限和设置服务状态

确认 config 文件位置

```console
$ psql -U postgres -c 'SHOW config_file'
```

修改postgresql.conf配置文件：

```
sudo vim /etc/postgresql/10/main/postgresql.conf
```

修改或添加下面一行：

```
listen_addresses = '*'
```

修改pg_hba.conf配置文件：

```
sudo vim /etc/postgresql/10/main/pg_hba.conf
```

修改或添加下面一行：

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             0.0.0.0/0               md5
```

保存文件，重启 postgresql 服务，并设置开机自动启动：

```
sudo systemctl restart postgresql
sudo systemctl enable postgresql
```

如果有防火墙，需要对应开启。

查看服务状态和关闭服务的命令分别为：

```
sudo systemctl status postgresql
sudo systemctl stop postgresql
```


### 添加主机端口映射

如果在 docker run 的时候没有指定端口映射，可以通过以下步骤添加

```console
# 查找容器 id
$ docker inspect some-container |grep Id
# 停止容器
$ docker stop some-container
# 停止 Docker 服务
$ systemctl stop docker
```

找到容器的配置文件目录

```console
cd /var/lib/docker/containers/some-container-id
```

修改 `hostconfig.json ` ， 将容器的端口映射到主机设备上的端口：

```json
"5432/tcp": [{"HostIp": "","HostPort": "5432"}]
```


修改 `config.v2.json` 在 `ExposedPorts` 中加入要暴露的端口：
```json
ExposedPoerts: {"5432/tcp":{}}
```

重启 Docker 后，外部设备就可以通过 主机IP:5432 访问应用了