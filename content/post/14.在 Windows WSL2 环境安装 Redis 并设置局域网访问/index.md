---
title: 在 Windows WSL2 环境安装 Redis 并设置局域网访问
slug: Installing-Redis-on-Windows-WSL2-and-setting-up-LAN-access
date: 2022-07-28 14:52:00+0000
image: cover.jpg
description: Redis 官方没有直接支持在 Windows 下安装运行
categories:
    - 开发环境
tags:
    - Windows
    - Redis
---

## 背景
需要多台电脑访问同一个本地的 redis 服务。打算在连接网线的固定IP的电脑上来安装 Redis，这台电脑是 Windows 10 系统，而 Redis 官方没有直接支持在 Windows 下安装运行。


下面是 [Redis 官方原话][2]：

> Use Redis on Windows for development Redis is not officially supported
> on Windows. However, you can install Redis on Windows for development
> by the following the instructions below.
> 
> To install Redis on Windows, you'll first need to enable WSL2 (Windows
> Subsystem for Linux). WSL2 lets you run Linux binaries natively on
> Windows. For this method to work, you'll need to be running Windows 10
> version 2004 and higher or Windows 11.

所以官方推荐安装在 [WSL2][3] 子系统上。

## 安装 WSL2
在 Windows 10 上安装 WSL2 很简单：
1. 打开 PowerShell(管理员) 
2. 运行命令
```bash
wsl --install
```
此命令将启用所需的可选组件，下载最新的 Linux 内核，将 WSL2 设置为默认值，并安装 Linux 发行版（默认安装 Ubuntu）。这个安装过程很快，期间会提示要重启一下电脑，电脑重启后会接着安装完成。

WSL2 安装完成后，可以在 PowerShell 用 `wsl` 命令直接进入 Linux 子系统。

## 安装和配置 Redis

### 安装 Redis：
```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis
```

### 配置 Redis 支持局域网访问：
修改 `/etc/redis/redis.conf`，注释掉 `bind 127.0.0.1`，并将 `protected-mode` 改为 `no`
```bash
# bind 127.0.0.1

protected-mode no
```

### 启动 Redis
```bash
sudo service redis-server start
```

## 从局域网访问 WSL2 网络
要实现从局域网访问 WSL2 网络，需要在 Windows 上配置 端口转发 和 防火墙允许入站规则。参考以下 PowerShell 命令（需以管理员权限执行）：
```bash
# 查询 WSL2 IP 地址
PS C:\Users\jony> wsl -- hostname -I
172.19.42.138

# 配置端口转发：外网访问 windows 8080 端口转发到 172.19.42.138:8080
PS C:\Users\jony> netsh interface portproxy add v4tov4 listenport=6379 connectaddress=172.19.42.138 connectport=6379

# 添加允许入站规则
PS C:\Users\jony> New-NetFirewallRule -DisplayName "Allow Inbound TCP Port 6379" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 6379
```
配置完后可在局域网用其他设备通过 Windows IP 访问 WSL2 服务。

对应的删除配置命令：
```bash
# 删除端口转发规则
C:\Users\jony> netsh interface portproxy delete v4tov4 listenport=6379

# 删除防火墙入站规则
C:\Users\jony> Remove-NetFirewallRule -DisplayName "Allow Inbound TCP Port 6379"
```

## 参考资料：

 - [Install Redis on Windows][4]
 - [使用 WSL 在 Windows 上安装 Linux][5]
 - [Windows 10 WSL2 网络配置][6]


  [1]: https://blog.niceue.com/usr/uploads/2022/07/1545570410.jpeg
  [2]: https://redis.io/docs/getting-started/installation/install-redis-on-windows/
  [3]: https://docs.microsoft.com/zh-cn/windows/wsl/
  [4]: https://redis.io/docs/getting-started/installation/install-redis-on-windows/
  [5]: https://docs.microsoft.com/zh-cn/windows/wsl/install
  [6]: https://ruihusky.github.io/ruihusky/posts/2020-12-11_wsl2-net-config/