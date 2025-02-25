---
title: 后续来啦：Winform/WPF中快速搭建日志面板
slug: follow-up-quickly-build-the-log-panel-in-wWinForm-or-wpf
description: 原理是将`Winform或WPF 应用程序，宿主到ASP.NET Core Web API`上
date: 2021-04-18 15:45:15
copyright: Default
originaltitle: 后续来啦：Winform/WPF中快速搭建日志面板
draft: False
cover: https://img1.dotnet9.com/2021/04/cover_02.jpg
categories: Winform,WPF
tags: Winform,WPF,日志面板,LogDashboard
---

继昨天发文`ASP.NET Core 可视化日志组件使用(`[阅读文章](https://mp.weixin.qq.com/s/BSt2hL32_vBMliu9-Fs1zg)，[查看视频](https://mp.weixin.qq.com/s/rbOeuS5PDEZRSHeRR9r8wA))后，视频下有朋友留言 **“Winform客户端的程序能用它不?”**，微信也有朋友问能否嫁接到`WPF`上，站长今早尝试了，是可以的！

![](https://img1.dotnet9.com/2021/04/0201.png)

原理是将`Winform或WPF 应用程序，宿主到ASP.NET Core Web API`上，具体先来个小视频看看效果，想要代码直接往下面翻：

【视频占位】

---

实战步骤：

1. 创建一个 WPF 应用程序
2. 添加`ASP.NET Core`、`Serilog`支持
3. WPF窗体中使用Serilog
4. 完结

---

**本文实战开始**

## 1. 创建一个 WPF 应用程序

使用VS 2019，创建一个`WPF 应用程序`项目，命名为**WPFWithLogDashboard**，本文基于`.NET 6搭建`。

## 2. 添加`ASP.NET Core`、`Serilog`支持

### 2.1 Nuget 安装相关Nuget包

`Microsoft.Extensions.Hosting`要指定版本，不能高于2.2.0:

```shell
Install-Package Microsoft.Extensions.Hosting -Version 2.2.0
Install-Package Serilog.AspNetCore
Install-Package LogDashboard
```

### 2.2 配置 Serilog 和 ASP.NET Core

打开`App.xaml.cs`文件，添加如下代码。仔细看，如下配置和上一篇文章 `Program.cs` 文件中的配置都是差不多的，主要是配置`Serilog`，记得输出日志分割符使用 **||**。

```C#
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;
using Serilog;
using Serilog.Events;
using System;
using System.Windows;

namespace WPFWithLogDashboard
{
	/// <summary>
	/// Interaction logic for App.xaml
	/// </summary>
	public partial class App : Application
	{
		protected override void OnStartup(StartupEventArgs e)
		{
			base.OnStartup(e);

			#region Serilog配置

			string logOutputTemplate = "{Timestamp:HH:mm:ss.fff zzz} || {Level} || {SourceContext:l} || {Message} || {Exception} ||end {NewLine}";
			Log.Logger = new LoggerConfiguration()
			  .MinimumLevel.Override("Default", LogEventLevel.Information)
			  .MinimumLevel.Override("Microsoft", LogEventLevel.Error)
			  .MinimumLevel.Override("Microsoft.Hosting.Lifetime", LogEventLevel.Information)
			  .Enrich.FromLogContext()
			  .WriteTo.Console(theme: Serilog.Sinks.SystemConsole.Themes.AnsiConsoleTheme.Code)
			  .WriteTo.File($"{AppContext.BaseDirectory}Logs/Dotnet9.log", rollingInterval: RollingInterval.Day, outputTemplate: logOutputTemplate)
			  .CreateLogger();

			#endregion

			Host.CreateDefaultBuilder(e.Args)
				.UseSerilog()
				.ConfigureWebHostDefaults(webBuilder =>
				{
					webBuilder.UseStartup<Startup>();
				}).Build().RunAsync();
		}
	}
}
```

添加`Startup.cs`文件，代码如下：

```C#
using LogDashboard;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Serilog;

namespace WPFWithLogDashboard
{
	public class Startup
	{
		private ILogger logger;
		public ILogger MyLoger
		{
			get
			{
				if (logger == null)
				{
					logger = Log.ForContext<Startup>();
				}
				return logger;
			}
		}

		public void ConfigureServices(IServiceCollection services)
		{
			MyLoger.Information("ConfigureServices");

			services.AddLogDashboard();
			services.AddControllers();
		}

		public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
		{
			MyLoger.Information("Configure");

			app.UseLogDashboard();

			app.UseRouting();

			app.UseEndpoints(endpoints =>
			{
				endpoints.MapControllers();
			});
		}
	}
}
```

在该文件中，主要作用是添加`LogDashboard`组件，配置`.NET CORE Web API`路由。

完成上面的代码，`Serilog`和`LogDashboard`两个组件其实已经安装、配置完成了:

1. 程序输出目录的**Logs**目录已经产生了日志文件。
2. 浏览器输入下面的链接，也能打开`LogDashboard`可视化日志面板了。

```shell
http://localhost:5000/logdashboard
```

## 3. WPF窗体中使用Serilog

主窗体`MainWindow.xaml`添加几个按钮，用于模拟添加普通日志、添加异常日志、打开可视化日志面板网页：

```XAML
<Window x:Class="WPFWithLogDashboard.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Windows中使用Serilog" Height="250" Width="350">
    <StackPanel Margin="20">
        <Button Content="打开日志面板" Width="100" Height="35" Click="OpenLogDashboard_Click"/>
        <Button Content="添加普通日志" Width="100" Height="35" Margin="0 20" Click="AddInfoLog_Click"/>
        <Button Content="添加异常日志" Width="100" Height="35" Click="AddErrorLog_Click"/>
    </StackPanel>
</Window>
```

`MainWindow.xaml.cs`中完成上面所说的功能：

```C#
using Serilog;
using System.Diagnostics;
using System.Windows;

namespace WPFWithLogDashboard
{
	public partial class MainWindow : Window
	{
		private ILogger logger;
		public ILogger MyLoger
		{
			get
			{
				if (logger == null)
				{
					logger = Log.ForContext<MainWindow>();
				}
				return logger;
			}
		}

		public MainWindow()
		{
			InitializeComponent();

			MyLoger.Information("WPF窗体中记录的日志");
		}
		private void AddInfoLog_Click(object sender, RoutedEventArgs e)
		{
			MyLoger.Information("测试添加Info日志");

		}

		private void AddErrorLog_Click(object sender, RoutedEventArgs e)
		{
			MyLoger.Error("测试添加异常日志");
		}

		private void OpenLogDashboard_Click(object sender, RoutedEventArgs e)
		{
			OpenUrl("http://localhost:5000/logdashboard");
		}

		private void OpenUrl(string url)
		{
			Process.Start(new ProcessStartInfo("cmd", $"/c start {url}")
			{
				UseShellExecute = false,
				CreateNoWindow = true
			});
		}
	}
}
```

OK，功能已经完成，本文基于`WPF`搭建的项目，也是适用于`Winform`项目模板的。

## 4. 完结

本文注重实践，如果您对`ASP.NET Core`不是很了解，建议您查看微软官方文档系统学习；不求甚解，直接Copy文中代码也成。

本文是否对您有用？记得3连走起哦。

> 
> 时间如流水，只能流去不流回。
> 
>- 公众号：Dotnet9
>- 作者、编辑：沙漠尽头的狼
>- 文中示例源码：https://github.com/dotnet9/TerminalMACS.ManagerForWPF/tree/master/src/Demo/WPFWithLogDashboard
>- 日期：2021-04-18
> 

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/04/2021-04-18_01.md)