---
title: (22/30)大家一起学Blazor：ASP.NET Core Identity(2)
slug: 22-of-30-let-us-learn-blazor-together-asp-dotnet-core-identity-2
description: 昨天做的验证只针对`Identity` 系统，没有包含到我们的日志
date: 2021-12-22 23:59:48
copyright: Reprint
author: StrayaWorker
originaltitle: (22/30)大家一起学Blazor：ASP.NET Core Identity(2)
originallink: https://ithelp.ithome.com.tw/articles/10269706
draft: False
cover: https://img1.dotnet9.com/2021/12/cover_05.png
albums: 学Blazor
categories: Blazor
tags: Blazor Server
---

昨天做的验证只针对`Identity` 系统，没有包含到我们的日志，如果在未登录状态下于地址栏输入`https://localhost:5018/Blog`，还是可以看到博客，让我们整合验证。

首先在`Blog.razor`外层加上`<AuthorizeView>`，这表示包在其中的内容呈现与否的条件为用户是否获得验证，接着在Blog内容外加上`<Authorized>`，顾名思义就是通过验证才能看到内容，另外新增一段未通过验证的`<NotAuthorized>`代码 。这边要记得加上`Context="IdentityContext"`，否则会跟Blog的`<EditForm>`本身的`context`产生冲突。

![](https://img1.dotnet9.com/2021/12/3301.png)

再去`App.razor`用`<CascadingAuthenticationState>`将原本的`<Router>`包裹，告诉`Blazor` 所有`Component` 都必须经过验证。

![](https://img1.dotnet9.com/2021/12/3302.png)

然后是`NavMenu.razor`，`<Authorized>`Component 加上注销链接，登录链接则移到`<NotAuthorized>`，icon 也改一下。

![](https://img1.dotnet9.com/2021/12/3303.png)

这时候要重新启动系统，验证机制才会生效，可以看到确实挡住了未验证访问。

![](https://img1.dotnet9.com/2021/12/3304.png)

在登录前，先打开`Dev Tool` 的`Application` 页签去看`Cookies`，目前只有一个`Cookie`。

![](https://img1.dotnet9.com/2021/12/3305.png)

登录后看到新的`Cookie`，这就是我们的`Authentication Cookie`。

![](https://img1.dotnet9.com/2021/12/3306.png)

接着来试试看注销，点击左边的`Logout`，并没有如想像中注销，而是来到一个Log out 页面，右上角还是登录状态，`Authentication Cookie` 也还在。

![](https://img1.dotnet9.com/2021/12/3307.gif)

我们去看`Logout.cshtml.cs`，里面有`OnGet()`(**站长注：.NET 6 Blazor server中没有该方法，请对比**)跟`OnPost()`分别对应`HttpGet`及`HttpPost`，我们从链接点过来的做法是`HttpGet`，但`OnGet()`这里什么事都没做，`OnPost()`则调用`ASP.NET Core Identity API` 将用户注销。

**原文原图：多了一个OnGet()方法**

![](https://img1.dotnet9.com/2021/12/3308.png)

**站长.NET 6项目截图：和上面对比，少了一个OnGet()方法**

![](https://img1.dotnet9.com/2021/12/3309.png)

`Logout.cshtml`也找到了一个`<form>`元素，`asp-page`这些`asp-`开头的代码是`ASP.NET Core` 的`Tag Helper` (标签协助代码)，类似`Angular` 的`*ngFor`，这边先略过不提，我们看到这里用了`method="post"`，还有个`<button type="submit">`，对应了刚才的`OnPost()`。

![](https://img1.dotnet9.com/2021/12/3310.png)

通常不会有用户点了注销后还要再点一次注销，所以我们这边改动一下，先给`<form>`加上`id="LogoutFormInLogout"`，再加上一段`<script>`，这里用到`IIFE(Immediately Invoked Functions Expressions)`，意即不需要调用就会执行的函式，一旦用户进入这页面就会将`<form>`提交，如此一来，只要点击`NavMenu`左边的Logout 链接，就可以顺利注销了。

![](https://img1.dotnet9.com/2021/12/3311.png)

如果不想要加上`id="LogoutFormInLogout"`这么长的id，也可以用`id="logoutForm"`就好，根目录下的`Pages/Shared/_LoginPartial.cshtml`里面有个注销的`<form>`已经定义了`id="logoutForm"`这个id。

![](https://img1.dotnet9.com/2021/12/3312.png)

不过现在注销后还停留在`Log out`页面似乎没有意义，所以将`OnPost()`的else区块改成`return LocalRedirect("~/Blog");`，这样注销后就会回到未验证的`Blog` 页面。

![](https://img1.dotnet9.com/2021/12/3313.png)

![](https://img1.dotnet9.com/2021/12/3314.gif)

这样一来就完成了单一项目的登录验证机制，而且各种功能、页面一应俱全，如果只是小型项目的话可以这么做，明天就来说明这些ASP.NET Core 的验证原理。

**引用：**

1. [立即函式IIFE](https://medium.com/vicky-notes/%E7%AB%8B%E5%8D%B3%E5%87%BD%E5%BC%8F-iife-27fe4007e446)
2. [Blazor cookie authentication Logout page](https://www.youtube.com/watch?v=pVaY7Th68U0&list=PL6n9fhu94yhVowClAs8-6nYnfsOTma14P&index=54)

**注：本文代码通过 .NET 6 + Visual Studio 2022重构，可点击原文链接与重构后代码比较学习，谢谢阅读，支持原作者**

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/12/2021-12-22_03.md)