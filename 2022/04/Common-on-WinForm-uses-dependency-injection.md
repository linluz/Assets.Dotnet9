---
title: 来，WinForm使用依赖注入！
slug: Common-on-WinForm-uses-dependency-injection
description: 关于依赖注入是什么？依赖注入是一种具体的编码技巧，对我来说最明显的就是解决代码的耦合性。
date: 2022-04-22 07:21:26
copyright: Reprint
author: AZRNG 鹏祥
originaltitle: 来，WinForm使用依赖注入！
originallink: https://mp.weixin.qq.com/s/murWKgo5KFMmMOJ4UQJWBQ
draft: False
cover: https://img1.dotnet9.com/2022/04/cover_29.jpg
categories: Winform
tags: Winform,依赖注入,IOC
---

>站长：Winform的分享文章不多，这个技术相对来说已经十分稳定了，该有的都有了，今天分享其他号写的Winform依赖注入一文，希望做相关开发的朋友能有新的体会。

## 1. 介绍

关于依赖注入是什么？依赖注入是一种具体的编码技巧，对我来说最明显的就是解决代码的耦合性。

## 2. 目的

ASP.NetCore中本身容器容器已经创建好了，只需要往容器添加服务即可，但是在Winform中默认还是通过new的方式来进行操作的(虽然我已经升级成了.Net6)，最近在把一个开源项目进行增加自用的功能，然后我顺带将原来的NetF升级为NetCore，然后就想用依赖注入方式去试试了。

>C/S代码写的少，如有不对，麻烦指正。

## 3. 操作

本文示例环境：VS2022、.Net6、Windows窗体应用

## 4. 准备

示例包含以下代码

- 窗体：Form1、Form2

- Service：IUserservice、Userservice、IOrderService、OrderService

```C#
public interface IUserservice
{
    string GetName();
}

public class UserService : IUserservice
{
    public string GetName()
    {
        return "IUserservice";
    }
}

public interface IOrderService
{
    string GetName();
}

public class OrderService : IOrderService
{
    public string GetName()
    {
        return "IOrderService";
    }
}
```

## 5. 场景

在Form1通过构造函数注入IUserservice，并且在Load事件里面调用IUserservice的获取名称方法，点击页面按钮后让Form2显示，Form2中通过依赖注入了IOrderService在Load事件里面调用IOrderService的获取名称方法。如果可以多次操作不报错就是成功。

## 6. 开始

引用组件

```xml
<PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="6.0.0" />
```

增加了一个ServiceProviderHelper的操作类

```C#
public static class ServiceProviderHelper
{
    /// <summary>
    /// 全局服务提供者
    /// </summary>
    public static IServiceProvider ServiceProvider { get; private set; } = null!;

    /// <summary>
    /// 初始化构建ServiceProvider对象
    /// </summary>
    /// <param name="serviceProvider"></param>
    /// <exception cref="ArgumentNullException"></exception>
    public static void InitServiceProvider(ServiceProvider serviceProvider)
    {
        ArgumentNullException.ThrowIfNull(serviceProvider, nameof(serviceProvider));

        ServiceProvider = serviceProvider;
    }

    /// <summary>
    /// 获取Form服务
    /// </summary>
    /// <param name="type"></param>
    /// <returns></returns>
    /// <exception cref="ArgumentException"></exception>
    public static Form GetFormService(Type type)
    {
        var service = ServiceProvider.GetRequiredService(type);
        if (service is Form fService)
        {
            return fService;
        }
        else
        {
            throw new ArgumentException($"{type.FullName} is not a Form");
        }
    }

    /// <summary>
    /// 获取Form服务
    /// </summary>
    /// <param name="type"></param>
    /// <returns></returns>
    /// <exception cref="ArgumentException"></exception>
    public static T GetService<T>() where T : class
    {
        return ServiceProvider.GetRequiredService<T>();
    }
}
```

修改Program方法

```C#
internal static class Program
{
    /// <summary>
    ///  The main entry point for the application.
    /// </summary>
    [STAThread]
    private static void Main()
    {
        //.net6写法 之前是三行合一行
        ApplicationConfiguration.Initialize();

        //创建服务容器
        var services = new ServiceCollection();
        //添加服务注册
        ConfigureServices(services);

        //构建ServiceProvider对象
        ServiceProviderHelper.InitServiceProvider(services.BuildServiceProvider());

        //获取指定服务
        var main = ServiceProviderHelper.ServiceProvider.GetRequiredService<Form1>();
        Application.Run(main);
    }

    /// <summary>
    /// 注入服务
    /// </summary>
    /// <param name="services"></param>
    public static void ConfigureServices(IServiceCollection services)
    {
        //批量注入可以使用Scrutor或者自己封装
        services.AddScoped<IUserservice, UserService>();
        services.AddScoped<IOrderService, OrderService>();

        //其他的窗体也可以注入在此处
        services.AddSingleton(typeof(Form1));
        services.AddTransient(typeof(Form2));
    }
}
```

分别在Form1和Form2进行注入

```C#
private readonly IUserservice _userservice;

public Form1(IUserservice userservice)
{
    InitializeComponent();
    _userservice = userservice;
}

private readonly IOrderService _orderService;

public Form2(IOrderService orderService) : this()
{
    _orderService = orderService;
}
```

点击Form1窗体按钮让Form2窗体显示

```C#
private void button1_Click(object sender, EventArgs e)
{
    var form2 = ServiceProviderHelper.GetFormService(typeof(Form2));
    form2.Show();
}
```

正常操作几次并没有发现异常。

## 7. 资料

在.NetCore3.1上基于Winform实现依赖注入实例：http://www.ty2y.com/study/znetcore3.1sjywinformsxylzrsl.html