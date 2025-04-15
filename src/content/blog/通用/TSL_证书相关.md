---
title: HTTPS 与 SSL 证书相关
description: ""
date: 2020-12-01
tags:
---

## HTTPS 简介

**超文本传输安全协议**（英语：HyperText Transfer Protocol Secure，缩写：HTTPS；常称为HTTP over TLS、HTTP over SSL或HTTP Secure）是一种通过计算机网络进行安全通信的传输协议。HTTPS经由HTTP进行通信，但利用SSL/TLS来加密数据包。HTTPS开发的主要目的，是提供对网站服务器的身份认证，保护交换资料的隐私与完整性。

## SSL 简介

**传输层安全性协议**（英语：Transport Layer Security，缩写：TLS）及其前身安全套接层（英语：Secure Sockets Layer，缩写：SSL）是一种安全协议，目的是为互联网通信提供安全及数据完整性保障。网景公司（Netscape）在1994年推出首版网页浏览器－网景导航者时，推出HTTPS协议，以SSL进行加密，这是SSL的起源。IETF将SSL进行标准化，1999年公布TLS 1.0标准文件（RFC 2246）。随后又公布TLS 1.1（RFC 4346，2006年）、TLS 1.2（RFC 5246，2008年）和TLS 1.3（RFC 8446，2018年）。在浏览器、电子邮件、即时通信、VoIP、网络传真等应用程序中，广泛使用这个协议。许多网站，如Google、Facebook、Wikipedia等也以这个协议来创建安全连线，发送资料。目前已成为互联网上保密通信的工业标准。

> 以上取自维基百科 [HTTPS](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE) [SSL](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A)

<!-- more -->

## 为网站配置 SSL 证书

### 首先获取 SSL 证书

SSL 证书颁发机构有很多，这里我们选择 [FreeSSL](https://freessl.cn/) 这个免费的证书颁发机构。

> 首先进行一系列的注册、验证邮箱、登录操作

**HTTPS 证书申请**

在输入完域名和邮件之后，下方会出现，我们就按默认的即可，它会让我们安装[KeyManager](https://keymanager.org/)，我们安装配置一下即可。

![](https://s3.ax1x.com/2020/12/01/DfAWIs.png)


在我们的域名解析处添加记录，主机记录为：_dnsauth；记录类型为：TXT；记录值为网站上显示的记录值。验证完成，我们继续下一步操作。直到从 KeyManager 中将证书导出。

### 配置 Nginx

首先查看 Nginx 是否安装有 `http_ssl_module` 模块

```
nginx -V
```

查看有 --with-http_ssl_module 字样即为装有该模块。

**配置文件**

```
server {
    listen 80 ssl;
    # ssl证书地址
    ssl_certificate     /*.crt;
    ssl_certificate_key  /*.key;
    # ssl验证相关配置
    ssl_session_timeout  5m;    #缓存有效期
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;    #加密算法
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;    #安全链接可选的加密协议
    ssl_prefer_server_ciphers on;   #使用服务器端的首选算法
    
    location / {
    }
}
```

```shell
# 重新加载nginx配置文件
nginx -s reload
```

这样我我们就可以通过 https 来访问我们的网站了。