---
layout: post
title: NSSM 自定义 Windows 服务
subtitle: 大大简化日常维护
tags: [FastAPI]
comments: true
---

## 简介

由于一些软件的需求，开发机装的是 Windows 系统。随着时间推移，运行的脚本越来越多，维护起来也越发头痛。这里学习用 NSSM 将脚本封装成服务，好处很多：

- 支持几乎所有程序（脚本或者.exe）
- 支持重定向输出（支持Rotation）
- 自动守护服务，程序挂掉可以自动重启
- 灵活的自定义环境变量

## 方法

1. 官网下载：[nssm.cc/download](nssm.cc/download)
2.  解压到合适的路径  
3. 在 `nssm.exe` 所在文件夹打开 `powershell`  
4. 填写相关内容，点击 `Install Service`  
5. `Win + R` 输入 `services.exe` ，即可方便地管理添加的服务  

## 示例

以如下 FastAPI 脚本(`main.py`)为例：

```python
from fastapi import FastAPI


@app.get("/")
def test():
    return {"code": "200", "msg": "Success"}


if __name__ == "__main__":
    uvicorn.run("main:app", host="127.0.0.1", port=8080, log_level="info")
```

在 `nssm.exe` 所在文件夹用 `powershell` 运行：

```powershell
nssm install
```

在 GUI中 填写：

**Path**: `python.exe` 目录  
**Startup directory**: FastAPI脚本所在目录  
**Argument**:  脚本名称(`main.py`)  
**Service name**: 服务名称，例如 MyApi

在 `Windows Services` 中把服务 MyApi 设为自动

## 服务管理

推荐使用 `Windows Services` 管理，或者也可以用 `nssm` 命令管理：

- 启动服务：`nssm start <servicename>`
- 停止服务： `nssm stop <servicename>`
- 重启服务:    `nssm restart <servicename>`

修改参数：`nssm edit <servicename>`

删除服务：`nssm remove <servicename>`  或 `nssm remove <servicename> confirm`    


ref：[【Windows学习】使用NSSM将exe封装为服务 by gtea](https://www.cnblogs.com/gtea/p/12672854.html)