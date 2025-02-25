---
title: (15/30)大家一起学Blazor：CSS isolation（隔离）
slug: 15-of-30-let-us-learn-blazor-together-css-isolation
description: 有时候会想对不同Component 做个别样式设置，但如果把class 都写在`wwwroot/css/site.css`，或是针对某个元素改动样式，可能导致改一个就影响全部Component，这种全域冲突是必须避免的，但应该怎么做？
date: 2021-12-18 22:35:17
copyright: Reprint
author: StrayaWorker
originaltitle: (15/30)大家一起学Blazor：CSS isolation（隔离）
originallink: https://ithelp.ithome.com.tw/articles/10264484
draft: False
cover: https://img1.dotnet9.com/2021/12/cover_05.png
albums: 学Blazor
categories: Blazor
tags: Blazor Server
---

## CSS isolation 介绍

有时候会想对不同Component 做个别样式设置，但如果把class 都写在`wwwroot/css/site.css`，或是针对某个元素改动样式，可能导致改一个就影响全部Component，这种全域冲突是必须避免的，但应该怎么做？

.NET 5 推出了CSS isolation(CSS 隔离)，建立Component 个别css 文件，命名规范为`{Component name}.razor.css`，文件会自动跟 `{Component name}.razor` 合并在一起，且名称不分大小写。下图可以看到不论是 `Blog.razor.css` 还是 `blog.razor.css` 都会跟 `Blog.razor` 被视为同一组。

![](https://img1.dotnet9.com/2021/12/2401.png)

![](https://img1.dotnet9.com/2021/12/2402.png)

CSS isolation 会在创建时被处理，Blazor 会改写CSS 选择器并产生一个 `{Project name}.style.css` 文件，可以在 `wwwroot/index.html (Blazor WebAssembly)` 或是 `Pages/_Layout.cshtml (Blazor Server)` 的 <head> 标签内找到引用路径，由于这是创建时才会产生，所以在项目内是看不到的，我们打开浏览器的Dev tool 并切换到Sources 页签，就能看到这文件。

![](https://img1.dotnet9.com/2021/12/2403.png)

![](https://img1.dotnet9.com/2021/12/2404.png)

有人可能会发现为何class 后面连接着没有看过的内容，例如 `.page[b-mxoy4q7bj7]` 或是`.main[b-mxoy4q7bj7]`，这是Blazor 用来识别CSS 选择器作用在哪个Component 的区域识别码，格式为`b-10 位数字及英文字母`，每个Component 的区域识别码都是独一无二的，可知这里的 `.main` 及`.page` class 只作用于 `[b-mxoy4q7bj7]` 对应的Component，注释写着`/* _content/BlazorServer/Shared/MainLayout.razor.rz.scp.css */`，打开`Shared/MainLayout.razor.css`，的确看到了 `.main` 及`.page` class，`rz.scp.css`附件名是用来识别CSS 选择器属于哪个Component。

![](https://img1.dotnet9.com/2021/12/2405.png)

我们在 `Blog.razor.css` 加上一段针对`label` 的样式修改，按下 `ctrl + shift + B` 生成项目，再看网页就能看到文字颜色改变了，`BlazorServer.style.css`也能看到 `Blog.razor.rz.scp.css` 的样式区块多了一段`label[b-0ae5hiw99t]`。

![](https://img1.dotnet9.com/2021/12/2406.png)

## 套用样式到Child Component

如果想对Post Component 的label 元素套用相同样式，又不想分别建立razor.css 文件呢？Blazor 提供了方便的做法，只要在CSS 选择器前面加上 `::deep` 即可，我们在 `Blog.razor.css` 的label 前面加上`::deep`，就能看到Post 的label 元素颜色都改变了，`BlazorServer.style.css`的class 也从 `label[b-0ae5hiw99t]` 变成了`[b-0ae5hiw99t] label`。

![](https://img1.dotnet9.com/2021/12/2407.png)

![](https://img1.dotnet9.com/2021/12/2408.png)

不过要注意的是，必须有父子关系 `::deep` 才能生效，Parent Compoent 的区域识别码仅作用于 `<div>` 标签，如果Child Component 没有被 `<div>` 标签包住就不会生效。我们把 `Blog.razor` 中包住Post Component 的 `<div>` 标签都注释，保存后再去网页看，发现 `<label>` 文字颜色都变回黑色了，但 `BlazorServer.style.css` 的class 仍旧是`[b-0ae5hiw99t] label`。

![](https://img1.dotnet9.com/2021/12/2409.png)

**引用：**

1. [ASP.NET Core Blazor CSS isolation](https://docs.microsoft.com/en-us/aspnet/core/blazor/components/css-isolation?view=aspnetcore-5.0)

**注：本文代码通过 .NET 6 + Visual Studio 2022重构，可点击原文链接与重构后代码比较学习，谢谢阅读，支持原作者**

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/12/2021-12-18_02.md)