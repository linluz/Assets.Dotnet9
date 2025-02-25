---
title: (3) MasaFramework 入门第三篇，使用MasaFramework
slug: Getting-Started-with-MasaFramework-Part-3-Using-MasaFramework
description: 使用MasaFramework
date: 2023-03-26 10:54:17
copyright: Reprint
author:  token的技术分享
originaltitle: (3) MasaFramework 入门第三篇，使用MasaFramework
originallink: https://www.cnblogs.com/hejiale010426/p/17227910.html
draft: false
cover: https://img1.dotnet9.com/2023/03/cover_09.png
categories: Web API
albums: MASA Framework
---

首先我们需要创建一个MasaFramework模板的项目，项目名称TokenDemo，项目类型如图所示：

![](https://img1.dotnet9.com/2023/03/1501.png)

删除Web/TokenDemo.Admin项目，新建Masa Blazor Pro项目模板项目，项目位置在src/Web项目：

![](https://img1.dotnet9.com/2023/03/1502.png)

项目类型选择ServerAndWasm，为了让我们支持俩种模式：

![](https://img1.dotnet9.com/2023/03/1503.png)

创建完成以后的目录，然后在TokenDemo.Admin中添加项目引用TokenDemo.Caller。

## 配置EntityFrameworkCore和Sqlite

修改TokenDemo.Service项目的包依赖为预览版

```xml
<ItemGroup>
    <PackageReference Include="Masa.BuildingBlocks.Dispatcher.Events" Version="1.0.0-preview.18" />
    <PackageReference Include="Masa.Contrib.Data.Contracts" Version="1.0.0-preview.18" />
    <PackageReference Include="Masa.Contrib.Data.EFCore.Sqlite" Version="1.0.0-preview.18" />
    <PackageReference Include="Masa.Contrib.Dispatcher.Events" Version="1.0.0-preview.18" />
    <PackageReference Include="Masa.Contrib.Dispatcher.IntegrationEvents.EventLogs.EFCore" Version="1.0.0-preview.18" />
    <PackageReference Include="FluentValidation" Version="11.5.1" />
    <PackageReference Include="FluentValidation.AspNetCore" Version="11.2.2" />
    <PackageReference Include="Masa.Utils.Extensions.DependencyInjection" Version="1.0.0-preview.18" />
    <PackageReference Include="Masa.Contrib.Service.MinimalAPIs" Version="1.0.0-preview.18" />
    <PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="6.0.3" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.5.0" />
</ItemGroup>
```
Masa.Contrib.Data.Contracts提供了[数据过滤](https://docs.masastack.com/framework/building-blocks/data/data-filter)的能力, 但它不是必须的，然后会出现报错，`LogMiddleware`将代码修改为以下代码：

```C#
namespace TokenDemo.Service.Infrastructure.Middleware;

public class LogMiddleware<TEvent> : EventMiddleware<TEvent>
    where TEvent : notnull, IEvent
{
    private readonly ILogger<LogMiddleware<TEvent>> _logger;

    public LogMiddleware(ILogger<LogMiddleware<TEvent>> logger)
    {
        _logger = logger;
    }

    public override async Task HandleAsync(TEvent action, EventHandlerDelegate next)
    {
        var typeName = action.GetType().FullName;

        _logger.LogInformation("----- command {CommandType}", typeName);

        await next();
    }
}
```

ValidatorMiddleware将代码修改为以下代码：

```C#
namespace TokenDemo.Service.Infrastructure.Middleware;

public class ValidatorMiddleware<TEvent> : EventMiddleware<TEvent>
    where TEvent : notnull, IEvent
{
    private readonly ILogger<ValidatorMiddleware<TEvent>> _logger;
    private readonly IEnumerable<IValidator<TEvent>> _validators;

    public ValidatorMiddleware(IEnumerable<IValidator<TEvent>> validators, ILogger<ValidatorMiddleware<TEvent>> logger)
    {
        _validators = validators;
        _logger = logger;
    }

    public override async Task HandleAsync(TEvent action, EventHandlerDelegate next)
    {
        var typeName = action.GetType().FullName;

        _logger.LogInformation("----- Validating command {CommandType}", typeName);

        var failures = _validators
            .Select(v => v.Validate(action))
            .SelectMany(result => result.Errors)
            .Where(error => error != null)
            .ToList();

        if (failures.Any())
        {
            _logger.LogWarning("Validation errors - {CommandType} - Command: {@Command} - Errors: {@ValidationErrors}", typeName, action, failures);

            throw new ValidationException("Validation exception", failures);
        }

        await next();
    }
}
```

OrderEventHandler将代码修改为以下代码：

```C#
namespace TokenDemo.Service.Infrastructure.Handlers;

public class OrderEventHandler
{
    readonly IOrderRepository _orderRepository;

    public OrderEventHandler(IOrderRepository orderRepository)
    {
        _orderRepository = orderRepository;
    }

    [EventHandler(Order = 1)]
    public async Task HandleAsync(QueryOrderListEvent @event)
    {
        @event.Orders = await _orderRepository.GetListAsync();
    }
}

public class OrderEventAfterHandler : IEventHandler<QueryOrderListEvent>
{
    public async Task HandleAsync(QueryOrderListEvent @event, CancellationToken cancellationToken = new CancellationToken())
    {
        await Task.CompletedTask;
    }
}
```

修改appsettings.json 添加Sqlite地址：

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=Catalog.db;"
  }
}
```

修改Program.cs代码：

```C#
using TokenDemo.Service.Infrastructure;

var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddAuthorization()
    .AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(options =>
    {
        options.Authority = "";
        options.RequireHttpsMetadata = false;
        options.Audience = "";
    });

builder.Services.AddMasaDbContext<ShopDbContext>(dbContextBuilder =>
{
    dbContextBuilder
        .UseSqlite() //使用Sqlite数据库
        .UseFilter(); //数据数据过滤
});

builder.Services.AddAutoInject();
var app = builder.Services
    .AddEndpointsApiExplorer()
    .AddSwaggerGen(options =>
    {
        options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme()
        {
            Name = "Authorization",
            Type = SecuritySchemeType.ApiKey,
            Scheme = "Bearer",
            BearerFormat = "JWT",
            In = ParameterLocation.Header,
            Description = "JWT Authorization header using the Bearer scheme. \r\n\r\n Enter 'Bearer' [space] and then your token in the text input below.\r\n\r\nExample: \"Bearer xxxxxxxxxxxxxxx\"",
        });
        options.AddSecurityRequirement(new OpenApiSecurityRequirement
        {
            {
                new OpenApiSecurityScheme
                {
                    Reference = new OpenApiReference
                    {
                        Type = ReferenceType.SecurityScheme,
                        Id = "Bearer"
                    }
                },
                new string[] {}
            }
        });
    })
    .AddFluentValidationAutoValidation().AddFluentValidationClientsideAdapters()
    .AddEventBus(eventBusBuilder =>
    {
        eventBusBuilder.UseMiddleware(typeof(ValidatorMiddleware<>));
        eventBusBuilder.UseMiddleware(typeof(LogMiddleware<>));
    })
    .AddServices(builder);

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.UseRouting();

app.UseAuthentication();
app.UseAuthorization();

app.UseHttpsRedirection();

app.Run();
```

添加EFCore迁移依赖：

```xml
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="6.0.15">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
</PackageReference>
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="6.0.15">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
</PackageReference>
```

在程序包管理器控制台中输入 `Add-Migration Init`生成Init的迁移文件（如果出现 `error NETSDK1082: Microsoft.AspNetCore.App 没有运行时包可用于指定的 RuntimeIdentifier“browser-wasm”`这个错误的话就先把`TokenDemo.Admin.WebAssembly`项目移除）。

使用`Update-Database`生成`Sqlitem`。

然后就可以看到项目中生成了`Catalog.db`文件。

启动TokenDemo.Service.Order我们就可以看到Swagger的界面了。

## 如何对接接口

打开TokenDemo.Caller项目中的`Callers\OrderCaller.cs`文件，修改`BaseAdderss`为`TokenDemo.Service.Order`的服务地址，打开`TokenDemo.Service.Order`项目的`Services\OrderService.cs`文件并且修改代码：

```C#
namespace TokenDemo.Service.Services;

public class OrderService : ServiceBase
{
    public OrderService(IServiceCollection services) : base(services)
    {
        App.MapGet("/order/list", QueryList).Produces<List<Infrastructure.Entities.Order>>()
            .WithName("GetOrders");
    }

    public async Task<IResult> QueryList(IEventBus eventBus)
    {
        var orderQueryEvent = new QueryOrderListEvent();
        await eventBus.PublishAsync(orderQueryEvent);
        return Results.Ok(orderQueryEvent.Orders);
    }
}
```

然后在通过命令行启动`TokenDemo.Service.Order`项目：

![](https://img1.dotnet9.com/2023/03/1504.png)

打开`TokenDemo\Admin`项目的`Pages\Home\Index.razor`文件并且修改代码：

```C#
@page "/"
@using TokenDemo.Caller.Callers
@inherits LayoutComponentBase
@inject NavigationManager Nav
@inject OrderCaller OrderCaller
@code {

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        Nav.NavigateTo(GlobalVariables.DefaultRoute,true);
        var data = await OrderCaller.GetListAsync();
        await base.OnAfterRenderAsync(firstRender);
    }

}
```

并且在`await base.OnAfterRenderAsync(firstRender);`这里打一个断点用于查看是否获取到消息，打开`TokenDemo.Admin.Server`项目的`Program.cs`，添加以下代码:

```C#
builder.Services.AddCaller(typeof(TokenDemo.Caller.Callers.OrderCaller).Assembly);
```

然后启动TokenDemo.Admin.Server项目，进入断点:

![](https://img1.dotnet9.com/2023/03/1505.png)

得到结果。

## 结尾

通过上文我们可以基本将MasaFramework的使用掌握，前端和后端的接口也掌握了。

当前是MasaFramework的第三篇入门，我会继续学习MasaFramework并且分享给大家。

来自token的分享。

- [MASA Framework](https://docs.masastack.com/framework/getting-started/overview)
- 学习交流：737776595