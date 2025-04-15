---
title: ABP 中是如何用 EF 实现仓储的
description: ""
date: 2020-11-19
tags: [.NET,ABP]
---

[ABP vNext 源码](https://github.com/abpframework/abp)

<!-- more -->

# ABP 中的仓储

## 仓储接口

先看 `IRepository` 仓储接口的定义，位置在 `Volo.Abp.Ddd.Domain/Volo/Abp/Domain/Repositories/IRepository.cs`

### IRepository

在 ABP 中所有的仓储都是直接或者间接继承自该接口，在ABP中仓储接口有两种类别

```csharp
interface IRepository<TEntity>
interface IRepository<TEntity, TKey>
```

这两种一种是标明主键类型的，另外一种是未标明主键类型。**标注主键类型的接口都会直接或间接继承自未标注主键类型的接口。**

### IBasicRepository

`IBasicRepository` 接口继承自 `IReadOnlyBasicRepository ` 定义基本的`CURD`操作

```csharp
interface IReadOnlyBasicRepository<TEntity> : IRepository where TEntity : class, IEntity
interface IReadOnlyBasicRepository<TEntity, TKey> : IRepository where TEntity : where TEntity : class, IEntity<TKey>
```

### IEfCoreRepository

下面我们看针对 EF Core 的仓储接口，位置在 `Volo.Abp.EntityFrameworkCore\Volo\Abp\Domain\Repositories\EntityFrameworkCore\IEfCoreRepository.cs`

```csharp
public interface IEfCoreRepository<TEntity> : IRepository<TEntity> where TEntity : class, IEntity
{
    DbContext DbContext { get; }
    DbSet<TEntity> DbSet { get; }
}

public interface IEfCoreRepository<TEntity, TKey> : IEfCoreRepository<TEntity>, IRepository<TEntity, TKey> where TEntity : class, IEntity<TKey>
{
}
```

没有做别的东西，定义了数据库上下文。

> 至此仓储接口也就这些，没有什么可看的。另外一些 `IReadOnlyBasicRepository` 这样的仓储接口最终都被 `IRepository` 继承无需多看。

---

## 仓储实现

### BasicRepositoryBase

`BasicRepositoryBase` 类完成 `IBasicRepositoryBase` 接口的抽象实现，比如

```csharp
public abstract Task<TEntity> InsertAsync(TEntity entity, bool autoSave = false, CancellationToken cancellationToken = default);
public abstract Task<TEntity> UpdateAsync(TEntity entity, bool autoSave = false, CancellationToken cancellationToken = default);
public abstract Task DeleteAsync(TEntity entity, bool autoSave = false, CancellationToken cancellationToken = default);
public abstract Task<List<TEntity>> GetListAsync(bool includeDetails = false, CancellationToken cancellationToken = default);
public abstract Task<long> GetCountAsync(CancellationToken cancellationToken = default);
public abstract Task<List<TEntity>> GetPagedListAsync(int skipCount, int maxResultCount, string sorting, bool includeDetails = false, CancellationToken cancellationToken = default);
```

`BasicRepositoryBase<TEntity>` 类拥有的方法操作

```csharp
Task<TEntity> InsertAsync(TEntity entity, bool autoSave = false, CancellationToken cancellationToken = default)
Task<TEntity> UpdateAsync(TEntity entity, bool autoSave = false, CancellationToken cancellationToken = default)
Task DeleteAsync(TEntity entity, bool autoSave = false, CancellationToken cancellationToken = default)
Task<List<TEntity>> GetListAsync(bool includeDetails = false, CancellationToken cancellationToken = default)
Task<long> GetCountAsync(CancellationToken cancellationToken = default)
Task<List<TEntity>> GetPagedListAsync(int skipCount, int maxResultCount, string sorting, bool includeDetails = false, CancellationToken cancellationToken = default)
```

`BasicRepositoryBase<TEntity, TKey>` 类拥有的方法操作(只列出它自身实现的未列出继承的) 

```csharp
Task<TEntity> GetAsync(TKey id, bool includeDetails = true, CancellationToken cancellationToken = default)
Task<TEntity> FindAsync(TKey id, bool includeDetails = true, CancellationToken cancellationToken = default)
Task DeleteAsync(TKey id, bool autoSave = false, CancellationToken cancellationToken = default)
```

> 注: 搞笑啊，`BasicRepositoryBase<TEntity, TKey>` 类竟然没有任何的引用，怎么寻思的。

### RepositoryBase

`RepositoryBase` 依旧是抽象类，完成的都是抽象实现，但他又继承了 `BasicRepositoryBase` 类，并完成 `IRepositoryBase` 接口的抽象实现。

`RepositoryBase<TEntity>`  类拥有的方法操作

```csharp
// 暂时未知他们是干什么用的
IDataFilter DataFilter { get; set; }
ICurrentTenant CurrentTenant { get; set; }
IAsyncQueryableExecuter AsyncExecuter { get; set; }
IUnitOfWorkManager UnitOfWorkManager { get; set; }

// 这三个没有被引用过
Type ElementType => GetQueryable().ElementType;
Expression Expression => GetQueryable().Expression;
IQueryProvider Provider => GetQueryable().Provider;

// 这两个都是直接调用 return GetQueryable();
IQueryable<TEntity> WithDetails()
IQueryable<TEntity> WithDetails(params Expression<Func<TEntity, object>>[] propertySelectors)

IQueryable<TEntity> GetQueryable();

Task<TEntity> FindAsync(Expression<Func<TEntity, bool>> predicate, bool includeDetails = true, CancellationToken cancellationToken = default);
Task<TEntity> GetAsync(Expression<Func<TEntity, bool>> predicate, bool includeDetails = true, CancellationToken cancellationToken = default)
    
// 应用数据过滤器 干啥用的?    
protected virtual TQueryable ApplyDataFilters<TQueryable>(TQueryable query) where TQueryable : IQueryable<TEntity>
```

`RepositoryBase<TEntity, TKey>` 类拥有的方法操作

```csharp
Task<TEntity> GetAsync(TKey id, bool includeDetails = true, CancellationToken cancellationToken = default)
Task<TEntity> FindAsync(TKey id, bool includeDetails = true, CancellationToken cancellationToken = default)
Task DeleteAsync(TKey id, bool autoSave = false, CancellationToken cancellationToken = default)
```

### EfCoreRepository

`EfCoreRepository` 是具体实现类，他重写抽象类 `RepositoryBase` 的所有方法。

先看 **EfCoreRepository<TDbContext, TEntity>** 类，看它的构造函数

```csharp
public EfCoreRepository(IDbContextProvider<TDbContext> dbContextProvider)
{
    _dbContextProvider = dbContextProvider;
    GuidGenerator = SimpleGuidGenerator.Instance;

    _entityOptionsLazy = new Lazy<AbpEntityOptions<TEntity>>(
        () => ServiceProvider
        .GetRequiredService<IOptions<AbpEntityOptions>>()
        .Value
        .GetOrNull<TEntity>() ?? AbpEntityOptions<TEntity>.Empty
    );
}
```

都干了什么呢?

- `dbContextProvider` 通过构造方法注入的方式获取到，用于获得 `DbContext` 。
  - `IDbContextProvider<TDbContext>` 的实现类是 `UnitOfWorkDbContextProvider<TDbContext>` 它的作用就是通过数据库连接串来创建 EF 的 `DbContext` 。

- `GuidGenerator` 获取一个用于生成 Guid 的工具类对象，实则就是对 `Guid.NewGuid();` 的调用。

- `_entityOptionsLazy` 他做了什么的呢? `Lazy` 延迟加载具体作用看介绍`Lazy`文章。`AbpEntityOptions` 类是干什么的? 未知。

**EfCoreRepository<TDbContext, TEntity, TKey>** 类，除了重写父类方法没做其他操作。

> 我们用的时候一般是从他开始用的，修改的话也是从他开始的，剩下的都是定义的基本的东西，用不到我们去修改操作。

