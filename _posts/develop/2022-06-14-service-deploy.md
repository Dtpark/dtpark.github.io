---
layout: post
title: 抖音项目笔记——项目部署方法 ｜ 青训营笔记
categories: develop
tags: go gin kitex deploy systemd
---

## 一、背景

不论项目采用微服务还是单体架构，开发完成后均需部署在服务器上。而项目在服务器上的部署与其在开发过程中调试要求有所不同。上线的项目要求稳定，调试的项目要求能够启动即可。故在部署上有所不同。本文将分享 Gin 框架在服务器上部署的方法。

## 二、思路

在调试 Gin 项目启动后，控制台会输出如下信息：

```bash
[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)
```
大意是 Gin 目前运行在调试模式，可以用环境变量或 go 语言命令切换到生产模式。所以部署要解决的第一个问题就是使 Gin 以生产模式运行。

以 ssh 方式连接到服务器终端通过设置环境变量成功启动后，还会存在问题。比如连接断开后，进程就会自动断开导致服务停止。避免因退出终端导致服务停止可以通过两种方法（笔者仅知道这两种）：

- 一是用窗口管理工具（**Tmux**, Screen等）新建一个窗口执行命令。这样不会因为退出终端导致窗口关闭，但是可能会因为系统原因导致进程挂掉，导致服务挂掉。

- 二是将运行命令写成一个 shell 脚本，通过将其封装成一个服务并配置自动重启实现服务高可用。

本文将分享第二种方法部署 Gin 的步骤。

## 三、步骤

1. 构建可执行文件

```bash
go build main.go
```
> 执行命令后就生成了可执行文件 main

2. 编写启动脚本 `run.sh`

```shell
#! /usr/bin/env bash
# 获取当前文件所在目录
CURDIR=$(cd $(dirname $0); pwd)
# 执行当前目录下的可执行文件 main 
exec "$CURDIR/main"
```

3. 在目录 `/usr/lib/systemd/system` 下封装服务文件 `xxx.service`

```
[Unit]
Description=micro_tiktok_gin

[Service]
Type=simple
Restart=always
RestartSec=3s
ExecStart=/root/micro_tiktok/cmd/api/run.sh

[Install]
WantedBy=multi-user.target
```

> - Description 服务描述
> - ExecStart 2. 中脚本绝对路径
> - WantedBy = multi-user.target 所有用户都可启动

4. 启动服务
```
systemctl start xxx.service
```

---

-   都看到这儿了，求个 star 呗~ [github.com/ByteDanceCa…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FByteDanceCamp%2Fmicro_tiktok "https://github.com/ByteDanceCamp/micro_tiktok")