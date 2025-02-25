---
title: (20/30)大家一起学Blazor：日志记录
slug: 20-of-30-let-us-learn-blazor-together-log-record
description: 在开发系统时，记录是一件很重要的事，前面都没有提到，笔者在最近才想到这点，所以就来实现吧！
date: 2021-12-21 23:36:29
copyright: Reprint
author: StrayaWorker
originaltitle: (20/30)大家一起学Blazor：日志记录
originallink: https://ithelp.ithome.com.tw/articles/10268616
draft: False
cover: https://img1.dotnet9.com/2021/12/cover_05.png
albums: 学Blazor
categories: Blazor
tags: Blazor Server
---

在开发系统时，记录是一件很重要的事，前面都没有提到，笔者在最近才想到这点，所以就来实现吧！

由于笔者用的是`Blazor Server`，官方文件提供的只有`Blazor WebAssembly` 的做法，所以先来试试看后者。

首先打开之前建立的`BlazorWasm` 项目，在`Counter.razor`加入`@using Microsoft.Extensions.Logging;`及注入服务`@inject ILogger<Counter> _logger;`，接着在原本的`IncrementCount()`内加入要提示的信息，这边用的是`LogWarning()`，除此之外还有`LogCritical`、`LogDebug`、`LogError`等等可以使用。

```html
@page "/counter"
@using Microsoft.Extensions.Logging
@inject ILogger<Counter> _logger

<PageTitle>Counter</PageTitle>

<h1>Counter</h1>

<p role="status">Current count: @_currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int _currentCount;

    private void IncrementCount()
    {
        _logger.LogWarning("有美女点击我了！");
        _currentCount++;
    }

}
```

接着将启动项目改成`BlazorWasm` 项目，启动后前往`Counter` 页面，点击按钮后，按下`F12` 切换到`Console` 页签，可以看到显示了我们定义的信息。

![](https://img1.dotnet9.com/2021/12/3001.gif)

在`Blazor WebAssembly` 这么简单，那在`Blazor Server` 也是一样吗？

可惜的是`Blazor Server` 并不支持这样的做法，目前只能用`IJSRuntime`的方式调用浏览器的`console.log`提示信息，想要有不同层级的信息也必须自己定制化。

我们切回`Blazor Server` 项目，在`JsInteropClasses`加入`ConsoleLog()`方法，里面做的事情就只有调用`console.log()`

```C#
public async Task ConsoleLog(string message)
{
	await _js.InvokeAsync<object?>("console.log", message);
}
```

接着在`Blog.razor.cs`覆写`OnAfterRenderAsync()`，在里面调用`ConsoleLog()`。

```C#
protected override async Task OnAfterRenderAsync(bool firstRender)
{
	await _jsClass!.ConsoleLog("这是Blazor Server的console.log信息");
}
```

![](https://img1.dotnet9.com/2021/12/3002.png)

之所以不写在`OnInitializedAsync()`是因为我们采用`预先渲染(pre-render)`的方式，此时的`JavaScript` 还没准备好，如果在`OnInitializedAsync()`调用，会发生下图的错误。

![](https://img1.dotnet9.com/2021/12/3003.png)

要是一定要在`OnInitializedAsync()`调用的话，可以去`_Layout.cshtml`将`<component>`的`render-mode`属性从`ServerPrerendered`改为`Server`。

![](https://img1.dotnet9.com/2021/12/3004.png)

Server 的`render-mode`分为三种：`Static`、`Server`及`ServerPrerendered`，第一种速度最快，将全部`Component`都转变为静态HTML 文件；第二种最慢，会先将一种标记传出，等到使用者启动该`Component` 后才会真的渲染成`HTML` 文件；第三种是折衷方案，先把`Component` 变成`静态HTML 文件`但没有交互功能，等到使用者启动该`Component` 后才会通知`Server` 将功能补上。

这也是为什么`render-mode`改成`Server`才有效的原因，因为此时的`ConsoleLog()`还没转成JavaScript 文件。

**引用：**

1. [ASP.NET Core Blazor logging](https://docs.microsoft.com/en-us/aspnet/core/blazor/fundamentals/logging?view=aspnetcore-5.0&pivots=server)
2. [LoggerExtensions Class](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.loggerextensions?view=dotnet-plat-ext-5.0)
3. [How can I write into the browser´s console via Blazor WebAssembly?](https://newbedev.com/how-can-i-write-into-the-browsers-console-via-blazor-webassembly)
4. [Blazor Server](https://stackoverflow.com/a/64814680)
5. [What's the difference between RenderMode.Server and RenderMode.ServerPrerendered in blazor?](https://stackoverflow.com/questions/58229732/whats-the-difference-between-rendermode-server-and-rendermode-serverprerendered)
6. [RenderMode Enum](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.rendermode?view=aspnetcore-5.0)

**注：本文代码通过 .NET 6 + Visual Studio 2022重构，可点击原文链接与重构后代码比较学习，谢谢阅读，支持原作者**

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/12/2021-12-21_02.md)