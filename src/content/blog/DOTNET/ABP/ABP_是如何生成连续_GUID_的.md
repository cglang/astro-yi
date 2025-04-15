---
title: ABP 是如何生成连续 GUID 的
description: ""
date: 2020-11-23
tags: [.NET,ABP]
---

## GUID 生成

GUID是数据库管理系统中使用的常见主键类型。并且 ABP 框架假定用户 ID 始终是 `GUID` 类型。

<!-- more -->

## IGuidGenerator 接口服务

GUID的最主要的问题是默认情况下它不是连续的。 当将 `GUID` 用作主键并将其设置为表的索引（默认设置）时，会在插入时带来严重的性能问题（因为插入新记录可能需要对现有记录进行重新排序）。

所以，不要使用 `Guid.NewGuid()` 为您的实体创建ID。ABP框架提供 `IGuidGenerator` 服务用于创建连续的 `GUID`。

```csharp
using System;
using System.Threading.Tasks;
using Volo.Abp.DependencyInjection;
using Volo.Abp.Domain.Repositories;
using Volo.Abp.Guids;

namespace AbpDemo
{
    public class MyProductService : ITransientDependency
    {
        private readonly IRepository<Product, Guid> _productRepository;
        private readonly IGuidGenerator _guidGenerator;

        public MyProductService(IRepository<Product, Guid> productRepository, IGuidGenerator guidGenerator)
        {
            _productRepository = productRepository;
            _guidGenerator = guidGenerator;
        }
        
        public async Task TestCreateGUIDAsync(string productName)
        {
            Guid guid = _guidGenerator.Create();
        }
    }
}
```

- IGuidGenerator.Create() 用于创建连续的 `Guid`。

## 配置

### AbpSequentialGuidGeneratorOptions

`AbpSequentialGuidGeneratorOptions` 是用于配置顺序GUID生成的选项类，它具有一个枚举类型属性。
    - `DefaultSequentialGuidType`（类型为 `SequentialGuidType` 的枚举）：生成GUID值时使用的策略。

数据库在处理 `GUID` 时会有所不同，因此您应根据指定数据库进行设置
    - `SequentialAtEnd` SQL Server
    - `SequentialAsString` MySQL or PostgreSQL
    - `SequentialAsBinary` Oracle

在模块的 `ConfigureServices` 中进行配置

```csharp
Configure<AbpSequentialGuidGeneratorOptions>(options =>
{
    options.DefaultSequentialGuidType = SequentialGuidType.SequentialAsBinary;
});
```

---

## 看源码

源码位置 `Volo.Abp.Guids\Volo\Abp\Guids\SequentialGuidGenerator.cs`

```csharp
public Guid Create(SequentialGuidType guidType)
{
    var randomBytes = new byte[10];
    RandomNumberGenerator.GetBytes(randomBytes);    // 先生成 10 字节的随机内容

    long timestamp = DateTime.UtcNow.Ticks / 10000L;// 获取当前时间戳的值 / 10000

    byte[] timestampBytes = BitConverter.GetBytes(timestamp);

    if (BitConverter.IsLittleEndian)
    {
        Array.Reverse(timestampBytes);
    }

    byte[] guidBytes = new byte[16]; // 用于存储最终生成 Guid 的数组

    switch (guidType)
    {
        // 可以看到 MySQL、PostgreSQL、Oracle 他们三个对 Guid 的处理方式是一样的
        case SequentialGuidType.SequentialAsString:
        case SequentialGuidType.SequentialAsBinary:

            Buffer.BlockCopy(timestampBytes, 2, guidBytes, 0, 6);
            Buffer.BlockCopy(randomBytes, 0, guidBytes, 6, 10);

            if (guidType == SequentialGuidType.SequentialAsString && BitConverter.IsLittleEndian)
            {
                Array.Reverse(guidBytes, 0, 4);
                Array.Reverse(guidBytes, 4, 2);
            }

            break;
        // SQL Server 独出一家
        case SequentialGuidType.SequentialAtEnd:

            Buffer.BlockCopy(randomBytes, 0, guidBytes, 0, 10);
            Buffer.BlockCopy(timestampBytes, 2, guidBytes, 10, 6);
            break;
    }

    return new Guid(guidBytes);
}
```

- 他的生成方式就是一个随机的 10 byte 的内容与当前时间 6 byte 相关的一个拼接，最终生成一个二进制长度为 128 位的 Guid。