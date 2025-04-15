---
title: ASP.NET Core MVC Razor页面在运行的时候进行编译
description: ""
date: 2020-11-16
tags: [.NET]
---

1. 安装 `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation` 程序包.
2. 更改 `StartUp` 类代码

```csharp
//services.AddControllersWithViews();
services.AddControllersWithViews().AddRazorRuntimeCompilation();
```


这样就完成了,简单记录一下.

参考

https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/view-compilation?view=aspnetcore-5.0&tabs=visual-studio

