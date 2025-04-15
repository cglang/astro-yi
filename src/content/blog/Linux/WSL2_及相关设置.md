---
title: 我使用的 WSL2 及相关设置
description: ""
date: 2020-12-24
tags:
---

## 简介

WSL 是 Windows Subsystem for Linux 的简称，他是一个基于 Windwos 子系统的 Linux，用于提供快速运行 Linux 命令和工具。WSL 目前有 WSL 和 WSL 2 两个版本，其中 WSL 2 功能更强大，并支持 Docker 的运行，这是我使用它的一个原因之一。

> [官方文档](https://docs.microsoft.com/zh-cn/windows/wsl/)

<!-- more -->

## 安装 WSL 并升级为 WSL2

> 可参考微软[官方文档](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10)

### 1. 启用适用于 Linux 的 Windows 子系统功能

在 `控制面板` - `程序和功能` - `启用或关闭 Windows 功能` - `适用于 Linux 的 Windows 子系统` ，勾选即可。

### 2. 在应用商店安装自己喜欢的 Linux 发行版

打开 `应用商店` - `搜索 'linux'` - `安装自己喜欢的发行版`

### 3. 使用 Linux

`开始菜单` - `启动安装的 Linux 发行版`

一般情况下第一次启动需要新建用户(我是用的是Ubuntu)，按提示进行即可。

### 4. 升级为 WSL 2

默认情况下启用该功能会是 WSL 版本而不是 WSL 2 版本，要使用 WSL 2 需手动升级。

在升级 WSL 2 之前需先**启用虚拟机功能**，因为 WSL 2 是在微软 Hyper-V 之上运行的。(以管理员身份在 PowerShell中执行下列命令)

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

**将 WSL 2 设置为默认版本**

```powershell
wsl --set-default-version 2
```

**重新启动** 计算机，以完成 WSL 2 更新。



## 开机启动 WSL

因为我会经常使用到 WSL 以及其中运行的服务，而且我不喜欢在每次启动电脑时手动的去启动 WSL 所以我做了以下操作。

### 打开 Windows 启动文件夹目录

按下 `Win + R` 输入 `shell:startup` ，新建一个 vbs 脚本文件，写入

```vbscript
Set ws = WScript.CreateObject("WScript.Shell")
ws.run "wsl -d Ubuntu-20.04"
```

我这里使用的是 `Ubuntu-20.04` ，你只需要换成自己的版本即可，使用 `wsl --list` 命令查看所有的 Linux 分发版本。这样在你启动计算机的时候指定的WSl Linux分发版就自动启动了

> 之所以使用 vbs 而不是批处理文件是因为我不想看见批处理文件的那个丑陋的黑框框。

> 注：WSL 在我的机器上大约会一直占用 1G 左右的内存，有时还会到 2G 左右，所以他是比较占内存的。



## 设置 Linux 系统自启动服务

WSL 的 Linux 是有很多坑的，就比方说他不会自动启动大不部分的服务，也不可使用 `systemctl` 命令。下面是我使用的解决方法。

**在WSL发行版中创建文件：/etc/init.wsl**，并设置其拥有可执行权限，比如我要开机启用 `supervisor` 服务，我只需如下操作。

```shell
sudo vim /etc/init.wsl

#!/bin/sh
service supervisor start
# 保存退出

sudo chmod +x /etc/init.wsl
```

然后我们编辑我们 Windwos 主机中自启动脚本，添加 ` -u root /etc/init.wsl` 参数。

```vbscript
Set ws = WScript.CreateObject("WScript.Shell")        
ws.run "wsl -d Ubuntu-20.04 -u root /etc/init.wsl"
```

> 参考文章 [WSL 服务自动启动的正确方法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/47733615)

## 限制 WSL2 占用系统资源

在用户目录下创建一个 `.wslconfig` 的文件

```
[wsl2]
processors=4
memory=2GB
swap=2GB
localhostForwarding=true
```

- processors 限制CPU的核数
- memory 限制内存使用大小
- swap 限制交换分区大小,一般设置与memory同样大小即可
- localhostForwarding 是否开启本地转发


## ~~宿主机连接 WSL 主机问题~~

> 在我刚使用 WSL2 的时候 `localhostForwarding` 这个功能会有 BUG 所以才使用了这个方法,现在本地转发可以正常使用不需要此解决方法了.

有时候我们在 WSL 中运行了一些服务，比如 MySQL 之类的我们需要使用它就要获取他的 IP ，但 WSL 每次启动的时候都会有不同的 IP 地址，所以每次重启计算机都要重新获取 WSL 的 IP 是一件很麻烦的事情，当然网上的解决方案有很多，就比如有使用脚本侥幸处理的，我使用的是名为 `go-wsl2-host` 的开源软件，GitHub 地址为 [go-wsl2-host](https://github.com/shayne/go-wsl2-host) ，可自行参考安装即可。

**我使用时遇到的问题**

1. 这个软件的方案就是修改主机的 hosts 文件来达到目的，如果有遇到**hosts**文件没有修改成功的问题，请尝试将 hosts 文件的只读属性取消勾选。



## WSL 中获取宿主机 IP 及自己的 IP

WSL 每次启动的时候都会有不同的 IP 地址，但他会把 IP 写在 `/etc/resolv.conf` 中。

**获取宿主机 IP**

```shell
cat /etc/resolv.conf | grep nameserver | awk '{ print $2 }'
```

**获取自己的 IP**

```shell
hostname -I | awk '{print $1}'
# 或者使用 ip 命令查看
ip a
```



## WSL 虚拟机的导出与导入

```shell
wsl --export <分发版名称> 导出文件.tar
wsl --import <分发版名称> <安装位置> 导入文件.tar
# 移除
wsl --unregister <分发版名称>
```



## WSL 1 升级为 WSL 2

### 查看 WSL 版本

```
wsl -l -v
wsl --list --verbose
```

### 下载 Linux 内核更新包

[更新包](https://learn.microsoft.com/zh-cn/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package)

### 设置默认Linux分发版的WSL版本

```
wsl --set-version {NAME} {1|2}
```
