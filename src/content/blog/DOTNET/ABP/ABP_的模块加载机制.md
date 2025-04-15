---
title: ABP 的模块加载机制
description: ""
date: 2020-11-22
tags: [.NET,ABP]
---

## 安装基础包

## 开始

首先新建一个解决方案添加两个项目，一个控制台项目(TestAbpConsole)，一个类库项目(Test)。在两个项目中都安装 `Volo.Abp.Core` 包。

<!-- more -->

**在 Test 项目中添加 `TestModule` 类**

```csharp
using Volo.Abp.Modularity;

namespace Test
{
    public class TestModule : AbpModule
    {
    }
}
```

**添加 `HelloWorldService` 类**

```csharp
using System;
using Volo.Abp.DependencyInjection;

namespace Test
{
    public class HelloWorldService : ITransientDependency
    {
        public void SayHello()
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

- `ITransientDependency` 是 ABP 的一个特殊接口, 它自动将服务注册为Transient 生命周期。

**修改 TestAbpConsole 项目的 Main 方法**

```csharp
using System;
using TestModule;
using Microsoft.Extensions.DependencyInjection;
using Volo.Abp;

namespace AbpModule
{
    class Program
    {
        static void Main(string[] args)
        {
            using var application = AbpApplicationFactory.Create<TestModule>();
            application.Initialize();
            var helloWorldService = application.ServiceProvider.GetService<HelloWorldService>();
            helloWorldService.SayHello();

            Console.ReadLine();
        }
    }
}
```

输出

```
Hello World
```

---

## 在 Web 项目中使用

使用 `abp` 命令新建一个 Web 项目

```
abp new Test
```

如果通过命令因为网络不好使请在[这个](https://abp.io/get-started#pills-profile)网站创建项目。

### 新建一个模块项目

新建一个名为 `Test` 的类库项目，首先通过 `NuGet` 工具安装 `Volo.Abp.AspNetCore.Mvc` 包，新建 `TestModule` 类

```csharp
using Microsoft.AspNetCore.Builder;
using Volo.Abp.Modularity;

namespace Test
{
    public class TestModule : AbpModule
    {
        public override void ConfigureServices(ServiceConfigurationContext context)
        {
        }

        public void Configure(IApplicationBuilder app)
        {
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(name: "default", pattern: "{controller=Home}/{action=Index}/{id?}");
            });
        }
    }
}
```

然后添加一个 `HomeController` 控制器

```csharp
using Microsoft.AspNetCore.Mvc;
using Volo.Abp.AspNetCore.Mvc;

namespace Test.Controller
{
    public class HomeController : AbpController
    {
        public IActionResult Index()
        {
            return Json("Hello World");
        }
    }
}
```

设置 `Test.Web` 项目为启动项，添加 `Test` 引用到此项目中， 在 `TestWebModule`  中添加模块

```csharp
[DependsOn(
    ...,
    typeof(TestModule)
)]
public class TestWebModule : AbpModule
{
...
```

先运行 `Test.DbMigrator` 完成数据库的迁移，然后启动 `Test.Web` 项目查看 `/Home/Index` 页面，可看到输出`Hello World`。
