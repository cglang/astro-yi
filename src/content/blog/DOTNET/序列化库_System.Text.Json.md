---
title: 序列化库 System.Text.Json
description: ""
date: 2020-11-24
tags: [.NET]
---

## 简介

`System.Text.Json` 组件是从 `.Net Core 3.0` 版本开始提供的，该组建的宗旨是提高序列化 `json` 字符串的效率。

<!-- more -->

## 正文

之前我有写过 [序列化与反序列化](https://cglang.github.io/2020/11/09/none/2020-11-09@a9bc03db-f053-4700-a0b5-337eaa098b7c/) 这个文章，其中用的是 `Newtonsoft.Json` 组件，现在我们可以使用 `System.Text.Json` 来替换掉他了。

```csharp
var names = new string[] { "Steve", "Alice", "张三", "李四" };

var jsonNames = JsonSerializer.Serialize(names);
Console.WriteLine(jsonNames);

names = JsonSerializer.Deserialize<string[]>(jsonNames);
foreach (var name in names)
    Console.Write(name + "\t");

Console.ReadKey();
```

---

## 会遇到的问题

### 输出的 json 中文被编码（乱码）的问题

**解决方法1**

在每次进行序列化时进行配置

```csharp
var names = new string[] { "Steve", "Alice", "张三", "李四" };

var options = new JsonSerializerOptions { Encoder = JavaScriptEncoder.Create(UnicodeRanges.All) };
var jsonNames = JsonSerializer.Serialize(names, options);
Console.WriteLine(jsonNames);
```

**解决方法2**

在 `Asp.Net Core` 中我们可以在 `services` 中进行配置

```csharp
services.AddControllersWithViews().AddJsonOptions(options =>
{
    options.JsonSerializerOptions.Encoder = JavaScriptEncoder.Create(UnicodeRanges.All);
});
// 或者是下面这两个
// services.AddControllers()
// services.AddMvc()
```



这样输出就不会有中文被编码的情况了

```
["Steve","Alice","张三","李四"]
```

### 格式化时间问题

```csharp
var times = new DateTime[] { DateTime.Now, DateTime.Now.AddDays(1) };
var jsonTimes = JsonSerializer.Serialize(times);
Console.WriteLine(jsonTimes);
```

输出

```
["2020-11-23T22:27:06.8656543+08:00","2020-11-24T22:27:06.8668047+08:00"]
```

你会发现它的格式为 `yyyy-MM-ddThh:mm:ss` 这样的格式，实际上在 `System.Text.Json` 中唯一支持的格式是 `ISO 8601-1:2019` 也就是刚才的那种格式。

一般我们前端使用的格式是 ``yyyy-MM-dd hh:mm:s` 或者 ``yyyy-MM-dd` 这样的，总之中间不加 "T" 这个字母

**解决方案**

新建 `DateTimeConverterUsingDateTimeParse` 类

```csharp
public class DateTimeConverterUsingDateTimeParse : JsonConverter<DateTime>
{
    public override DateTime Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        Debug.Assert(typeToConvert == typeof(DateTime));
        return DateTime.Parse(reader.GetString());
    }

    public override void Write(Utf8JsonWriter writer, DateTime value, JsonSerializerOptions options)
    {
		writer.WriteStringValue(value.ToString("yyyy-MM-dd"));
    }
}
```

```csharp
JsonSerializerOptions options = new JsonSerializerOptions();
options.Converters.Add(new DateTimeConverterUsingDateTimeParse());

var times = new DateTime[] { DateTime.Now, DateTime.Now.AddDays(1) };
var jsonTimes = JsonSerializer.Serialize(times, options);
Console.WriteLine(jsonTimes);
```

输出

```
["2020-11-23","2020-11-24"]
```

**如果是 Asp.Net Core 应用程序** 我们可以 `startup` 中使用 `services` 配置全局

```
services.AddControllers().AddJsonOptions(options =>
{
	options.JsonSerializerOptions.Converters.Add(new DateTimeConverterUsingDateTimeParse());
});
```



---

更多问题请参考

https://docs.microsoft.com/zh-cn/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to?pivots=dotnet-5-0

