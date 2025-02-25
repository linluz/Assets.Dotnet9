---
title: (16/30)大家一起学Blazor：建立数据库
slug: 16-of-30-let-us-learn-blazor-together-create-database
description: 我们现在有了基本的日志，但是每次输入完重新加载页面数据都会重置，因为这些数据都只存在于浏览器，没有真正储存到数据库，为了保存下来，我们要跟数据库连接。
date: 2021-12-18 23:34:26
copyright: Reprint
author: StrayaWorker
originaltitle: (16/30)大家一起学Blazor：建立数据库
originallink: https://ithelp.ithome.com.tw/articles/10265408
draft: False
cover: https://img1.dotnet9.com/2021/12/cover_05.png
albums: 学Blazor
categories: Blazor
tags: Blazor Server
---

我们现在有了基本的日志，但是每次输入完重新加载页面数据都会重置，因为这些数据都只存在于浏览器，没有真正储存到数据库，为了保存下来，我们要跟数据库连接。
(注：Blazor WebAssembly 没有直接跟数据库交互的能力，不过微软有提供`ASP.NET Core hosted` 选项，可以在建立Blazor WebAssembly 时一并建立ASP.NET Core Web API 工程)

## Entity Framework Core

首先在`appsettings.json`加入连接字串，因为测试用，所以SqlServer 用本地数据库。

```json
{
  "ConnectionStrings": {
    "DBConnection": "Server=(localdb)\\MSSQLLocalDB;database=Blog;integrated security=true;" 
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

接着 NuGet 下载两个组件，分别为`Microsoft.EntityFrameworkCore.SqlServer` 跟`Microsoft.EntityFrameworkCore.Tools`，这两个组件是跟SqlServer 交互用的ORM 组件，ORM 又是什么呢？

**项目工程配置文件：BlazorServer.csproj**

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

	<PropertyGroup>
		<TargetFramework>net6.0</TargetFramework>
		<Nullable>enable</Nullable>
		<ImplicitUsings>enable</ImplicitUsings>
	</PropertyGroup>

	<ItemGroup>
	  <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="6.0.1" />
	  <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="6.0.1">
	    <PrivateAssets>all</PrivateAssets>
	    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
	  </PackageReference>
	</ItemGroup>

</Project>
```

远古时代的程序开发者想从`SqlServer` 取数据回来，都必须学习`SQL 语句`，后来`.NET` 平台(包括`VB`、`C#`)开发了`ADO.NET` 这个取数据的好工具，将数据取回来后一一跟程序`mapping`，但`ADO.NET` 在数据映射上不如人意，微软便推出了`Entity Framework`，这就是初始的`ORM(Object-Relational Mapper)`，让开发者可以把数据变成`实体(entity)`，不用实际去接触`SQL 语句`，且有了设计工具(Designer) 让程序开发者更好操作。

但也有些人觉得`Entity Framework` 太笨重就转向另一个轻便的工具`Dapper`的怀抱，笔者也用过`Dapper`，如果是喜欢自己组`SQL 语句`的人确实会觉得方便。

而`.NET Core` 这个跨平台版本的`ORM` 就是用`Entity Framework Core` 去做数据映射的处理。

安装完组件后，新增一个类 `AppDbContext`，继承`DbContext`，里面的构造函数将`DbContextOptions<AppDbContext> options`传给基类 也就是`DbContext`。

**AppDbContext.cs**

```C#
using Microsoft.EntityFrameworkCore;

namespace BlazorServer.Models;

public class AppDbContext : DbContext
{
	public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
	{
	}

	public DbSet<BlogModel>? Blogs { get; set; }

	public DbSet<PostModel>? Posts { get; set; }
}
```

接着打开`Program.cs`，注册使用数据库，这边用的是`UseSqlServer()`，当然也有`UseMysql()`、`UseOracle()`可以用，只要安装相应的组件即可。

![](https://img1.dotnet9.com/2021/12/2501.png)

打开`程序包管理器控制台`，输入命令`Add-Migration Init`，告诉`Entity Framework Core` 要建立一个数据库迁移，接着就会产生迁移目录`Migration`，这是`Entity Framework Core` 的特点，在程序跟数据库之间产生中介`Migration`，另外`Migration` 一大好处就是在真正确定前，都可以不断修改，等确定后再更新数据库。

![](https://img1.dotnet9.com/2021/12/2502.png)

![](https://img1.dotnet9.com/2021/12/2503.png)

## Relation between Blog and Post

接下来要在`Post` 里面加上`BlogId`，`Blog` 对于`Post `而言是`主表`之于`子表`，如果新增迁移的时候没有加入对主表的关联(也就是主表外键关联子表)，EF Core 很聪明会主动加上，但字段名就会是`Model+Key` 的名称，变成`BlogModelBlogId`，我们来把名称简化。

![](https://img1.dotnet9.com/2021/12/2504.png)

先在`PostModel`加上两个属性，`BlogId`跟`Blog`，切记一定要这样添加，`BlogId`是存在数据库用的，`Blog`则是在关联数据的时候有个实体可以用，让我们不用自己join 数据表，后续会再说明。

![](https://img1.dotnet9.com/2021/12/2505.png)

接着用`Remove-Migration`将原先的Migration 清除，再新增一次Migration，即执行命令`Add-Migration Init`，可以看到新的Migration 有简化的名称`BlogId`了，这时候再去更新数据库。

![](https://img1.dotnet9.com/2021/12/2506.png)

使用命令`Update-Database`去更新数据库，然后打开`SQL Server 对象资源管理器`，可以找到刚才建立的数据库`Blog`，数据库名来自连接字串`database`的名称，可以看到数据库已经建起来了，过程中没有用到SQL 语句或是SSMS 界面。

![](https://img1.dotnet9.com/2021/12/2507.png)

![](https://img1.dotnet9.com/2021/12/2508.png)

不过有一点要特别注意，中途如果换数据库的话，原先的Migration 有很大机率产生问题，各家数据库的数据类型都有差异，所以最好一开始就规划好用哪个数据库。

除了`AddDbContext<T>`这种最常见的做法，还有`AddDbContextFactory<T>`这种在个别组件产生新的`DbContext`的方法，因为`AddDbContext<T>`的生命周期是`scoped`，对Blazor Server 来说也就是除非关闭系统，否则`DbContext`都不会dispose，有些人希望生命周期仅限于组件即可，就可以用`AddDbContextFactory<T>`。

**引用：**

1. [Entity Framework](https://zh.wikipedia.org/wiki/Entity_Framework)
2. [Dapper](https://www.nuget.org/packages/Dapper/)
3. [Database Providers](https://docs.microsoft.com/en-us/ef/core/providers/?tabs=dotnet-core-cli)
4. [New DbContext instances](https://docs.microsoft.com/en-us/aspnet/core/blazor/blazor-server-ef-core?view=aspnetcore-5.0#new-dbcontext-instances-1)

**注：本文代码通过 .NET 6 + Visual Studio 2022重构，可点击原文链接与重构后代码比较学习，谢谢阅读，支持原作者**

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/12/2021-12-18_03.md)