---
title: "【缓存】Redis 基于 WSL 在 Windows 上的安装"
date: 2022-05-25
tags: ["缓存", "redis", "win"]
description : "该篇文章主要是介绍怎么使用 WSL 在 WIN10 上的安装"
---
该篇文章主要是介绍怎么使用 WSL 在 WIN10 上的安装

## 前提
Win10 在本机已经打开 WSL2(Windows Subsystem for Linux) ,  在  Microsoft Store 安装 Unbuntu, 安装完后会让你输入 用户名和密码。然后就可以 linux 命令行了。 WSL2 启动为 `控制面板\程序\程序和功能\` 左侧的 “启动或关闭 Windows 功能“ 中的 ”适用于 Linux 的 Windows 子系统“。打上勾就好了。
![开启 WSL](/images/post/cache/WSL2.png)

## 安装 redis
使用以下命令进行安装 redis
```bash
sudo apt-add-repository ppa:redislabs/redis
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install redis-server
```

启动 redis
```bash
sudo service redis-server start
```

## 检测是否安装成功

在 Linux 的交互界面上

```bash
redis-cli
127.0.0.1:6379> ping
PONG
```
表示已经安装成功了。

## 可以使用开源客户端
[AnotherRedisDesktopManager](https://github.com/qishibo/AnotherRedisDesktopManager)

## 迁移
由于之前在商城上下载都默认安装在 C 盘，如果这样下去 C 盘容量会不保。因此需要迁移它的位置。
### 停止
先停止在运行的 wsl, 不然会报无权限的错误。
```shell
wsl --shutdown
```
## 导出文件
1. 使用命令 `wsl -l` 查看需要导出的文件

```
适 用 于 Linux 的 Windows 子 系 统 分 发 版 :
Ubuntu-20.04 (默 认 )
```

2. 导出需要导出的文件
```shell
wsl --export Ubuntu-20.04 F:\WSL\export.tar
```
Ubuntu-20.04 : 子系统的分发版
`F:\WSL\export.tar` ：目标路径

## 导入文件
导入需要的文件
```
wsl --import kaito-Ubuntu F:\WSL\Ubuntu F:\WSL\export.tar
```
- `kaito-Ubuntu` : 分发版名称
- `F:\WSL\Ubuntu` : 安装目录
- `F:\WSL\export.tar` : 导入的文件

导入后默认的用户名为 root

再次使用 `wsl -l` 就会出现两个了。这时需要使用命令 `wsl --setdefault kaito-Ubuntu` 来更换默认的子系统, 如果觉得没必要遗留之前的，就可以注销掉之前安装的子系统，注销的命令为  `wsl --unregister Ubuntu-20.04`

这时候使用命令 `wsl` 时就变成了 `kaito-Ubuntu` 了。启动 redis 服务就可以使用了。

参考链接：
- [Install Linux on Windows with WSL](https://docs.microsoft.com/en-us/windows/wsl/install)
- [install-redis-on-windows](https://redis.io/docs/getting-started/installation/install-redis-on-windows/)