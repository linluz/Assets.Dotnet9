---
title: WPF版本问题排坑记录
slug: WPF-version-issue-troubleshooting-record
description: 珍爱生命，远离不明第三方组件库。
date: 2023-04-17 20:00:29
copyright: Contribution
author: 一位极少露面的靓仔
originaltitle: WPF版本问题排坑记录
originallink: https://zhuanlan.zhihu.com/p/622610149
draft: false
cover: https://img1.dotnet9.com/2023/04/cover_02.png
categories: WPF
---

> 本文由网友投稿。
>
> 作者：一位极少露面的靓仔
>
> 原文标题：WPF版本问题排坑记录
>
> 原文链接：https://www.cnblogs.com/akwkevin/p/17288814.html

本文由网友投稿，文中示例仓库：[Dotnet9WPFControls](https://github.com/dotnet9/Dotnet9WPFControls)，使用介绍：[WPF|快速添加新手引导功能（支持MVVM）](https://dotnet9.com/22/5/Wpf-quickly-add-newbie-guide-support-MVVM)，示例正常运行扣1，失败扣0，并留下吐嘈....

原文如下：

---

先说结论：**珍爱生命，远离不明第三方组件库**【站长：。。。】

## 问题描述

今早摸鱼的时候看见狼哥一个开源项目[Dotnet9WPFControls](https://github.com/dotnet9/Dotnet9WPFControls)，我非常感兴趣！结果clone下来之后，没跑起来？嗯？我姿势不对？好！我再跑！结果还是run不起来，无奈求教狼哥本人，狼哥亲自clone之后没在他的机器上重现我这个bug。因为报错的这个库是nuget拉下来的，我们先看报错截图：

![](https://img1.dotnet9.com/2023/04/0201.png)

## 排坑之旅

第三方库是："MaterialDesignThemes.Wpf”

问题已经很明显了，是因为库的依赖出现的程序集版本不对所引起的，我们只需要找到这个PresentationFramework然后找到相应版本就行了，问题似乎很清晰明了。

好的，我们按照上述思路，开始操作：

先查询当前所引用的PresentationFramework在本地机器上的存储位置：

![](https://img1.dotnet9.com/2023/04/0202.png)

于是我们进入此文件目录：

![](https://img1.dotnet9.com/2023/04/0203.png)

一开始并没有6.0.2和6.0.16这个文件夹，这是后面我解决这个问题所下载的。

然后我们点进文件夹，发现:

![](https://img1.dotnet9.com/2023/04/0204.png)

这个PresentationFramework就是报错的6.0.0版本，于是按照我们上述的解决思路，我们只需要将这个替换成6.0.2版本问题不就解决了吗？

## 新的问题来了

于是说干就干，但新的问题来了，6.0.2版本的PresentationFramework我去哪里找？

我观察了下PresentationFramework的其他几个类似dll，发现这个6.0.0是和 .net sdk的版本挂钩的，也就是说我下载net6.0.2的sdk就可以找到PresentationFramework6.0.2版本。

于是前去微软官网下载：

![](https://img1.dotnet9.com/2023/04/0205.png)

这里下载请注意，我经过测试，下载右边的桌面运行时是没用的，必须下载左边的。

## 解决他

下载完后，我们怎么替换引用呢？

很简单，我想到了一个很狗但是非常方便的方法（因为我只是想运行这个项目，不涉及release所以可以这样做，但如果你需要release请务必禁止这样做！）

没错，就是文件重命名，下载完6.0.2版本后我们的引用文件夹长这样：

![](https://img1.dotnet9.com/2023/04/0206.png)

我们来个狸猫换太子！

把6.0.2改成6.0.0：

![](https://img1.dotnet9.com/2023/04/0206.png)

最后，重启项目，完美解决并成功运行：

![](https://img1.dotnet9.com/2022/05/5209.gif)

![](https://img1.dotnet9.com/2022/05/5210.gif)

## 总结

站长这也是新电脑环境，运行正常，欢迎留言讨论。