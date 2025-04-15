---
title: HTTP 协议
description: ""
date: 2021-04-30
tags:
---

## HTTP

> HTTP: HyperText Transfer Protocol 超文本传输协议

## 协议概述

HTTP是一个`客户端`和`服务端`之间请求和应答的标准，通常使用 TCP 协议。客户端和服务端是相对来说的，在这里客户端可以通指发起 HTTP 请求的一端，服务端为处理并响应HTTP请求的一端。HTTP 是无状态的，即客户端不做请求操作服务端无法知道客户端的存在状态。

> 在 HTTP 0.9 和 1.0 中，每一次请求都会与服务端进行 TCP 连接，等服务端处理并响应完请求之后就会断开连接，下一次请求时会重新进行 TCP 连接操作，称为`短连接`。在 HTTP 1.1 中，引入了保持连线的机制(默认使用此功能)，一个连接可以重复在多个请求/回应使用。持续连线的方式可以大大减少等待时间，因为在发出第一个请求后，双方不需要重新运行 TCP 握手程序，称为`长连接`。

<!-- more -->

## 请求报文构成

HTTP 报文由三部分组成: `请求行` `请求头` `请求体`

### 请求行

```
[请求方法] [URL] [HTTP协议及版本]

GET /home/index.html HTTP/1.1
```

### 请求头

请求头是一组键值对的集合，下面是我请求 Baidu 页面是的部分请求头。

```
Bdpagetype: 1
Bdqid: 0x9dc4c5be00000aa7
Cache-Control: private
Connection: keep-alive
Content-Encoding: gzip
Content-Type: text/html;charset=utf-8
date: Fri, 30 Apr 2021 03:01:20 GMT
```

### 请求体

请求体可以包含大量信息的其他信息，通常是将表单组件中的值通过编码成 `param1=value1&param2=value2` 形式的字符串进行传输。

### 一个完整的请求报文

如下

```
POST /api/user HTTP/1.1
Host:localhost
Connection:keep-alive
Content-Type:application/x-www-form-urlencoded
Content-Lenght:16

name=name&age=22
```

## 响应报文构成

响应报文同请求报文类似，也分为三部分 `响应行` `响应头` `响应体`

### 响应行(状态行)

```
[HTTP协议及版本] [状态码] [状态码短语]
HTTP/1.1 200 OK
```

### 响应头

响应头也是一组键值对的集合

```
date:Fri, 30 Apr 2021 03:27:53 GMT
Content-Type:application/json; charset=utf-8
Server:Kestrel
Transfer-Encoding:chunked
```

### 响应体

响应体内包含本次请求服务端响应的资源。

### 一个完整的响应体


```
HTTP/1.1 200 OK
date:Fri, 30 Apr 2021 03:27:53 GMT
Content-Type:application/json; charset=utf-8
Server:Kestrel
Transfer-Encoding:chunked

{"value":"1223"}
```

## 请求方法(动作)

HTTP/1.1协议中共定义了八种方法（也叫“动作”）来以不同方式操作指定的资源:

### GET

向指定的资源发出“显示”请求。使用GET方法应该只用在读取资料，而不应当被用于产生“副作用”的操作中，例如在网络应用程序中。其中一个原因是GET可能会被网络爬虫等随意访问。参见安全方法。浏览器直接发出的GET只能由一个url触发。GET上要在url之外带一些参数就只能依靠url上附带querystring。

### HEAD

与GET方法一样，都是向服务器发出指定资源的请求。只不过服务器将不传回资源的本文部分。它的好处在于，使用这个方法可以在不必传输全部内容的情况下，就可以获取其中“关于该资源的信息”（元信息或称元数据）。

### POST

向指定资源提交数据，请求服务器进行处理（例如提交表单或者上传文件）。数据被包含在请求本文中。这个请求可能会创建新的资源或修改现有资源，或二者皆有。每次提交，表单的数据被浏览器用编码到HTTP请求的body里。浏览器发出的POST请求的body主要有两种格式，一种是application/x-www-form-urlencoded用来传输简单的数据，大概就是"key1=value1&key2=value2"这样的格式。另外一种是传文件，会采用multipart/form-data格式。采用后者是因为application/x-www-form-urlencoded的编码方式对于文件这种二进制的数据非常低效。

### PUT

向指定资源位置上传其最新内容。

### DELETE

请求服务器删除Request-URI所标识的资源。

### TRACE

回显服务器收到的请求，主要用于测试或诊断。

### OPTIONS

这个方法可使服务器传回该资源所支持的所有HTTP请求方法。用'*'来代替资源名称，向Web服务器发送OPTIONS请求，可以测试服务器功能是否正常运作。


### CONNECT

HTTP/1.1协议中预留给能够将连接改为隧道方式的代理服务器。通常用于SSL加密服务器的链接（经由非加密的HTTP代理服务器）。

## 响应状态码

- 1xx 消息——请求已被服务器接收，继续处理
- 2xx 成功——请求已成功被服务器接收、理解、并接受
- 3xx 重定向——需要后续操作才能完成这一请求
- 4xx 请求错误——请求含有词法错误或者无法被执行
- 5xx 服务器错误——服务器在处理某个正确请求时发生错误

---

# 用于维持 HTTP 状态的机制

基于 HTTP 的请求都是无状态的，HTTP 只是作为一个传输协议的存在，于是为了解决无状态问题有了下面的机制。

## Cookie

> 大部分的客户端(如浏览器)都实现了 Cookie 机制

### Cookie的工作原理

浏览器(客户端)在第一次发送请求到服务端的时候，服务端会创建 Cookie，Cookie 中包含一些用户相关的信息用于标识客户端，然后会将 Cookie 发送到客户端，客户端再次请求服务端时将会携带 Cookie，服务端通过 Cookie 来区分和标识不同的客户端。

由于 Cookie 中的信息客户端可以肆意进行修改，所以对服务器来说 Cookie 中的信息不是完全可信的，为了解决这个问题就有了 Session ，他是基于 Cookie 的。



### Session

浏览器(客户端)第一次发送请求到服务器端时，服务器端会创建一个 Session 对象，同时会创建一个特殊的 Cookie 值(name 为JSESSIONID, value 为 Session 对象的 Id)，如:`JSESSIONID:948a620034259042545e3f3a469b20f9`，然后将该 Cookie 发送至客户端。

客户端每次请求都会携带 Cookie , Cookie 中存有 Session 对象的 Id ，服务端接到请求之后会通过 SessionId 获取到 Session 对象，进而可以获取到 Session 中存储的信息。由于存储在 Cookie 中的 SessionId 是由服务端产生的，客户端无法进行伪造，所以对服务端而言 Session 中存储的信息就是可信的。

一般情况下 Session 对象的存活时间为 30 分钟，过期之后服务端会自动销毁对象，当客户端再次请求时服务端将会重新创建新的对象。当客户端在存活时间内携带 SessionId 进行请求时，服务端会刷新 Session 对象的存活时间。


## 区别

主要区别是 Cookie 数据保存在客户端，Session 数据是保存在服务器中的，具体客户端或者服务端将数据保存在哪里就要看各自的实现机制了。比如浏览器客户端会将 Cookie 存储到硬盘当中，并可以为其设置过期期限；服务器端可以将 Session 数据存储到各种数据库或者是内存当中，一般情况下的实现会直接存储到内存当中。

> 在 ASP.NET Core 中只提供了 ISession 接口，具体实现需要自己选择，可以是 `内存`、`Redis 数据库`、`SQL 数据库` 等任何方式。

---

> Cookie 与 Session 并不属于 HTTP 标准的一部分，他是为了用于维持 HTTP 状态的机制，他会因为不同的实现而有所区别。所以直接说 Cookie 大小限制这样的话是错误的，不过你可以问某一浏览器某一版本对 Cookie 大小的限制是多少。