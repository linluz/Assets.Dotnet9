---
title: 如何在 Prism 的 Module 中弹出对话框？
slug: how-to-pop-up-a-dialog-box-in-prism-module
description: 有网友提出需求，在Prism的Module中如何弹出对话框？像主界面弹出关于对话框一样？
date: 2021-04-14 17:32:16
copyright: Default
originaltitle: 如何在 Prism 的 Module 中弹出对话框？
draft: False
cover: https://img1.dotnet9.com/2021/04/cover_04.jpg
categories: WPF
tags: WPF,Prism,Module
---

>有网友提出需求，在Prism的Module中如何弹出对话框？像主界面弹出关于对话框一样？

![网友需求](https://img1.dotnet9.com/2021/04/0401.png)

关于窗口弹出效果：

<video id="video" controls="" preload="none" poster="https://img1.dotnet9.com/2021/04/0403.png">
  <source id="mp4" src="https://img1.dotnet9.com/2021/04/0402.mp4" type="video/mp4">
</video>

效果一般，如果有设计师可以做的很漂亮。

本文不打算写的很详细，具体做法可以看源码（文末附仓库链接），另外关于Prism的学习可参考如下文章：

1. NET Core 3.x WPF MVVM框架Prism系列(后续版本通用)

```shell
https://dotnet9.com/11338.html
```

2. WPF企业级开发框架搭建指南（启示录），2020从入门到放弃

```shell
https://jhrs.com/2020/37391.html
```

## 1 关于对话框如何弹出来的？

### 1.1 通用对话框窗体注册

App.xaml.cs

```C#
protected override void RegisterTypes(IContainerRegistry containerRegistry)
{
  ...
  containerRegistry.RegisterDialogWindow<DialogWindow>();//注册自定义对话框窗体
  ...
}
```

如上代码，`RegisterDialogWindow`为注册对话框窗体代码，即所有弹出的对话框是包裹在`DialogWindow`内的，代码简单我就不贴了，知道他是一个`Window`就好，具体请看源码：[DialogWindow](https://github.com/dotnet9/lqclass.com/blob/main/src/LQClass.AdminForWPF/LQClass.CustomControls/Controls/DialogWindow.xaml)。

### 1.2 实际对话框注册

也是在App.xaml.cs文件中，下面的代码即为注册实际的对话框代码：`QQ群对话框`、`关于对话模式`、`推荐网站对话框`等，`括号中的字符串(如"About")`用于弹出对话框定位使用，**注：实际的对话框是基于`UserControl`编写的，只有这样才能套入是一个`Window`的`RegisterDialogWindow`内**。

```C#
protected override void RegisterTypes(IContainerRegistry containerRegistry)
{
  ...
  containerRegistry.RegisterDialogWindow<DialogWindow>();//注册自定义对话框窗体
  containerRegistry.RegisterDialog<QQGroupView, QQGroupViewModel>("QQGroup");
  containerRegistry.RegisterDialog<AboutView, AboutViewModel>("About");
  containerRegistry.RegisterDialog<WebView, WebViewModel>("Web");
  ...
}
```

### 1.3 菜单触发调出对话框

这个就更简单了，直接在xaml中使用，比如主窗口的关于菜单：

```html
<MenuItem
    Command="{Binding RaiseShowDialogCommand}"
    CommandParameter="About"
    Header="{markup:I18n {x:Static i18NResources:Language.About}}">
    <MenuItem.Icon>
        <Path
            Width="16"
            Data="{StaticResource InfoGeometry}"
            Fill="{DynamicResource SuccessBrush}" />
    </MenuItem.Icon>
</MenuItem>
```

`RaiseShowDialogCommand`在VM的基类`ViewModelBase`定义的，实际调用代码如下：

```C#
/// <summary>
/// 显示对话框
/// </summary>
/// <param name="dlgName"></param>
private void RaiseShowDialogHandler(string dlgName)
{
  DialogService.ShowDialog(dlgName);
}
```

参数`dlgName`即上面`1.2`注册对话框时指定的字符串，视情况指定。

## 2 Module中的对话框如何弹出？

这一节就是本文的重点了。

## 2.1 关键：IModule的RegisterTypes方法

`Module`是动态加载的，模块中的对话框不可能写在`App.xaml.cs`中噻？如果你对`Prism`比较熟悉，看看各模块的`Module`入口类，即继承`IModule`的模块入口类方法`RegisterTypes(IContainerRegistry containerRegistry)`，可以在这里注入模块使用的类型和对话框，供本模块或者其他模块使用。

## 2.2 模块对话框实现

以`LQClass.ModuleOfLog`模块为例，我们在下面注册添加日志对话框：

```C#
public void RegisterTypes(IContainerRegistry containerRegistry)
{
  ...
  containerRegistry.RegisterDialog<AddView, AddViewModel>("AddLogView");
  ...
}
```

`AddView`为添加日志视图，和一般用户控件没有区别，可参考关于对话框编写，本项目视图链接见[AddView.xaml](https://github.com/dotnet9/lqclass.com/blob/main/src/LQClass.AdminForWPF/Modules/LQClass.ModuleOfLog/Views/AddView.xaml)。

## 2.3 怎么调用？

调用方式同主窗口菜单一致，代码文件路径：`src\LQClass.AdminForWPF\Modules\LQClass.ModuleOfLog\Views`，`添加`按钮声明如下：

```XAML
<Button
  Margin="10,15,10,0" 
  Style="{StaticResource ButtonSuccess}"
  Command="{Binding RaiseShowDialogCommand}"
  CommandParameter="AddLogView"
  Content="{markup:I18n {x:Static i18NResources:Language.Add}}"/>
```

命令参数`AddLogView`即`2.2`中注册视图时给的名称。

效果如下，只是演示模块中如何弹出对话框，未加实际业务：

<video id="video" controls="" preload="none" poster="https://img1.dotnet9.com/2021/04/0404.png">
  <source id="mp4" src="https://img1.dotnet9.com/2021/04/0405.mp4" type="video/mp4">
</video>

## 3 关于项目怎么运行？

**说明：本项目服务端基于[WTM](https://wtmdoc.walkingtec.cn/)搭建，客户端由WPF编写。**

### 3.1 运行服务端

如果需要运行查看客户端效果，请先编译服务端`src\LQClass.Admin`，至于怎么编译，不会的自行百度哦，编译成功后，调试运行服务端项目，或者双击运行服务端Exe：`src\LQClass.Admin\LQClass.Admin\bin\Debug\net5.0\LQClass.Admin.exe`，服务端默认开启5000端口。

<video id="video" controls="" preload="none" poster="https://img1.dotnet9.com/2021/04/0406.png">
  <source id="mp4" src="https://img1.dotnet9.com/2021/04/0407.mp4" type="video/mp4">
</video>

### 3.2 运行WPF客户端

如果只是看源码，可以忽略本步骤。

注意查看`App.config`，客户端连接服务端的地址是否正确：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
	<appSettings>
    ...
		<add key="API" value="http://localhost:5000/api/"/>
	</appSettings>
  ...
</configuration>
```

项目路径：`src\LQClass.AdminForWPF`，生成成功，调试客户端项目，或者双击客户端Exe：`\src\LQClass.AdminForWPF\Build\LQClass.AdminForWPF.exe`即可运行本客户端了。

后端默认管理员账号：

```shell
admin
000000
```

最后来个完整演示：

<video id="video" controls="" preload="none" poster="https://img1.dotnet9.com/2021/04/0408.png">
  <source id="mp4" src="https://img1.dotnet9.com/2021/04/0409.mp4" type="video/mp4">
</video>

> 
> 时间如流水，只能流去不流回。
> 
>- 公众号：Dotnet9
>- 号主微信号：dotnet9com
>- 作者、编辑：沙漠之尽头的狼
>- 乐趣课堂仓库（欢迎提issue）：https://github.com/dotnet9/lqclass.com
>- 日期：2021-04-14
> 

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/04/2021-04-14_01.md)