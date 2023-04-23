---
layout: post
title: 让局域网内其他用户访问应用 
tags: [VMWare]
comments: true
---

在 VMWare Player 安装 Linux 系统虚拟机，将虚拟机内 Docker 容器的应用映射到虚拟机端口（参考[这里]()）后，主机一般可以通过类似 `192.168.*.*:port` 的地址访问 Docker 容器内的应用  
但有时候，我们希望同一局域网内的其他机器也能通过主机 IP + 端口号访问这个应用，这时需要进行一些配置  
VMWare Player 作为精简版本，没有提供可视化的设置界面，但我们依然可以通过一些设置实现目的  
首先，查询虚拟机的 IP 地址
```
$ ip addr
```
关闭虚拟机，右键打开虚拟机的设置，在网络设置中将网络模式设为 `NAT` 模式 

找到 VMWare 的网络配置文件 `vmnetnat.conf`  

例如 Win 10 中的路径为 `C:\ProgramData\VMware\vmnetnat.conf`  

备份后以管理员权限打开这个文件，我们需要在 `[incomingtcp]` 项目中添加相应内容，格式为：

```
主机端口号 = 虚拟机IP地址:虚拟机端口号
```  

例如，将虚拟机的 `192.168.101.128:8888` 转发到主机 `8888` 端口：
```txt
8888 = 192.168.101.128:8888
```
我们还可以顺手设置将虚拟机的 `22` 端口转发到主机的特定端口（例如 `30022`），这样就可以便捷的通过主机+特定端口号直接访问虚拟机了

```txt
30022 = 192.168.101.128:22
```

最终这部分的配置文件类似于：

```txt
[incomingtcp]
# Use these with care - anyone can enter into your virtual machine through these...

# FTP (both active and passive FTP is always enabled)
#      ftp localhost 8887
#8887 = 192.168.101.128:21

# WEB (make sure that if you are using named webhosting, names point to
#     your host, not to guest... And if you are forwarding port other
#     than 80 make sure that your server copes with mismatched port 
#     number in Host: header)
#      lynx http://localhost:8888
#8888 = 192.168.27.128:80
8888 = 192.168.101.128:8888

# SSH
#      ssh -p 8889 root@localhost
30022 = 192.168.101.128:22
```

请务必注意安全问题，配置文件也提示我们，任何人都可以通过主机的端口访问到虚拟机，请在可信任的网络环境中进行上述操作

接下来，我们需要在 Windows 防火墙中放行端口，具体方法就不作赘述了

最后，我们重启 `VMWare NAT Service` 以确保配置生效

以管理员身份运行 `cmd` 并运行以下命令

```cmd
net stop "VMWare NAT Service"
net start "VMWare NAT Service"
```

启动虚拟机和 Docker 应用后，我们应该可以通过 `主机IP:8888` 访问应用，也可以通过 `ssh 虚拟机用户名@主机IP -p 30022` 直接访问虚拟机了

关于如何将虚拟机内 Docker 容器的应用映射到虚拟机端口的操作，请参考[这里]()