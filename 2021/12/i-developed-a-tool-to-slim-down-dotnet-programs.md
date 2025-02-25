---
title: 我开发了一个对.NET程序进行瘦身的工具
slug: i-developed-a-tool-to-slim-down-dotnet-programs
description: 我开发了一个对.Net程序瘦身的工具，可以把被引用但是没有被使用的程序集删除。我用它把一个.Net core程序从`147兆`瘦身到`59.5`兆。
date: 2021-12-26 21:56:27
copyright: Reprint
author: 杨中科
originaltitle: 我开发了一个对.NET程序进行瘦身的工具
originallink: https://mp.weixin.qq.com/s/B7QdVQWtgBKNKOEFj2Lg4g
draft: False
cover: https://img1.dotnet9.com/2021/12/cover_43.png
categories: .NET相关
tags: .NET,瘦身
---

我开发了一个对.Net程序瘦身的工具，可以把被引用但是没有被使用的程序集删除。我用它把一个.Net core程序从`147兆`瘦身到`59.5`兆。

![](https://img1.dotnet9.com/2021/12/cover_43.png)

.NET中发布程序的时候有对程序集进行剪裁的功能，但是那个功能只能做静态检查。比如我们的项目使用了A程序集，A程序集中的类有M1、M2两个方法，M1方法中又调用了B程序集的代码，M2方法中调用了C程序集的代码。如果我们的程序中只调用了M1方法，而没有调用M2方法，那么用.NET的剪裁是不能把没有被调用的M2方法中的调用的C程序集剪裁掉的。

我的这个工具可以做运行时检查，会把在运行时完全没有被调用（会考虑到反射等动态机制）的程序集删除掉。.NET中发布程序的程序集剪裁功能也不支持WinForm、WPF项目。

我的这个工具的实现原理并不复杂，但是我找了一圈都没有找到类似软件，所以就自己写了一个。大家如果知道有这样的工具，请告诉我，如果确认这是我的首创的话，我会把这个软件完善（测试各种项目和.NET版本以及各个操作系统的兼容性）后发布并开源。

如果这个项目开源的话，我会发布到我的自媒体，各位朋友可以关注我的哔哩哔哩、今日头条、抖音、微博、油管等频道，频道名都是“杨中科”。

>原文作者：杨中科
>
>原文链接：https://mp.weixin.qq.com/s/B7QdVQWtgBKNKOEFj2Lg4g

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/12/2021-12-26_01.md)