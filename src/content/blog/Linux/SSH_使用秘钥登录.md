---
title: SSH 使用秘钥登录
description: ""
date: 2020-12-07
tags:
---

## 配置 SSH 连接免密码登录

<!-- more -->

### 生成秘钥对

使用 `ssh-keygen` 命令生成秘钥对

**常用的命令**

```shell
ssh-keygen -f "file_name"
ssh-keygen -t rsa -b 4096 -C "your_email@domain.com" -f "file_name"
```

### 为远程主机配置公钥

**1. 使用 `ssh-copy-id` 命令**

```shell
ssh-copy-id remote_username@server_ip_address
```

**2. 手动配置**

1. 首先先连接远程主机。
2. 使用文本编辑工具打开 `~/.ssh/authorized_keys`
3. 将生成的 `*.pub` 公钥文件的内容复制到 `authorized_keys` 文件内，可以配置多个公钥，每行一个公钥。

### 使用私钥远程连接主机

```shell
ssh -i "private_key" "user_name@server_ip"
```

## 可能会遇到的问题

###  权限问题

`authorized_keys` 文件需要严格的权限，查看此文件的权限是不是 `600`，不是的话重新设置。

```shell
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
```

### ssh 配置文件问题

老版本的 ssh 需要进行配置，配置 `/etc/ssh/sshd_config`，将下列两项取消注释。

```
AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
```

**普通非root用户需要开启此项**

```
PubkeyAuthentication yes
```

### 秘钥对生成问题

windows 下生成的秘钥对会有可能不是 openssh 格式需要将之转为 openssh 格式秘钥。
如果可行的话通过远程主机进行秘钥对生成。

