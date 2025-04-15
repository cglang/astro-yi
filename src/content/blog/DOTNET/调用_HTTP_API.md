---
title: .NET 中调用 HTTP API
description: ""
date: 2021-04-18
tags: [.NET]
---

当我们需要在 .NET 中调用 `RestAPI` 的时候我们可以使用 `HttpWebRequest` `WebClient` `HttpClient`，我来描述一下这三种的使用以及区别。


<!-- more -->

## WebRequest

首先先讲一下 `WebRequest` ，`WebRequest` 是一个`抽象类`, 是一种基于特定的 http 实现,在处理 Request 时底层会根据传入的 url 生成相应的子类.如: `HttpWebRequest` `FileWebRequest`. `HttpWebRequest` 是一种相对底层的处理 Http request/response 的方式。我们可以通过此类来存取 `headers` `cookies` 等信息.

### 在获取响应时指定为 HttpWebResponse

```csharp
WebRequest webRequest = WebRequest.Create(uri);
webRequest.Method = "GET";
HttpWebResponse webResponse = (HttpWebResponse)webRequest.GetResponse();
```

### 在创建时指定为 HttpWebRequest 

```csharp
HttpWebRequest http = (HttpWebRequest)WebRequest.Create("http://baidu.com");
http.Method = "GET";
WebResponse response = http.GetResponse();
```

### 读取响应内容

```csharp
Stream stream = response.GetResponseStream();
StreamReader streamReader = new StreamReader(stream);
string data = streamReader.ReadToEnd();
```

## WebClient

`WebClient` 提供了对 `HttpWebRequest` 的高层封装，用来简化调用。在不需要精细化配置的情况下可以进行使用。比方说使用它来设置请求时的 `Cookie` 就需要以字符串的形式直接写入 `Headers` 当中,而不能操作 `Cookie` 类型的对象.

```csharp
using (var webClient = new WebClient())
{
    string data = webClient.DownloadString("http://baidu.com");
}
```

### 写入 Cookie

```csharp
using (WebClient webClient = new WebClient())
{
    webClient.Encoding = Encoding.GetEncoding("utf-8");
    webClient.Headers.Add("Content-Type", "application/json");
    webClient.Headers.Add(HttpRequestHeader.Cookie, $@"{CookieName}={CookieValue}");
    byte[] responseData = webClient.UploadData(url, "POST", postData);
    re = JsonConvert.DeserializeObject<JObject>(Encoding.UTF8.GetString(responseData));
}
```



## HttpClient

`HttpClient` 是一种新的处理 Http request/response 工具包，具有更高的性能。他是在 .NET Framework 4.5 中被引入的，它吸取了 `HttpWebRequest` 的灵活性及 `WebClient` 的便捷性，它所有关于 IO 操作的方法都是异步的。


```csharp
HttpClient client = new HttpClient();
HttpResponseMessage response = await client.GetAsync("http://baidu.com");
if (response.IsSuccessStatusCode)
{
    string data = await response.Content.ReadAsStringAsync();
}
```

默认情况下 `HttpClient` 不会自动抛出异常, 通过调用 `response.EnsureSuccessStatusCode();` 可以在请求失败的时候抛出异常，当有需要的时候可以判断 `response.StatusCode` 的值手动抛出异常。

另外每次 `Request` 就实例化一次 `HttpClient`，大量的请求必会将 socket 耗尽抛出 `SocketException` 异常。正确的使用方法应该是使用单例模式或工厂类来创建 `HttpClient` 对象。

在 ASP.NET Core 中我们可以下列方式获取 `HttpClient` 对象。

```csharp
// 添加服务
services.AddHttpClient();

...

// 在构造函数中注入 IHttpClientFactory 类
private readonly IHttpClientFactory httpClientFactory;

public TestController(IHttpClientFactory httpClientFactory)
{
    this.httpClientFactory = httpClientFactory;
}

...

// 使用 IHttpClientFactory 工厂获取 HttpClient 对象
HttpClient httpClient = httpClientFactory.CreateClient("RequertBaidu");
HttpResponseMessage httpResponse = await httpClient.GetAsync("http://baidu.com");
if (httpResponse.IsSuccessStatusCode)
{
    string data = await httpResponse.Content.ReadAsStringAsync();
}
```

参考: [如何在 ASP.NET Core 中使用 HttpClientFactory ?](https://mp.weixin.qq.com/s/Cd46cNnWIb_wt8vlO-Joeg)