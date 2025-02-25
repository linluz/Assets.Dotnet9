---
title: (12/30)大家一起学Blazor：Cascading values and parameters
slug: 12-of-30-let-us-learn-blazor-together-cascading-values-and-parameters
description: 昨天不小心把Reset按钮的type改成button，今天改回reset。
date: 2021-12-15 23:38:24
copyright: Reprint
author: StrayaWorker
originaltitle: (12/30)大家一起学Blazor：Cascading values and parameters
originallink: https://ithelp.ithome.com.tw/articles/10262797
draft: False
cover: https://img1.dotnet9.com/2021/12/cover_05.png
albums: 学Blazor
categories: Blazor
tags: Blazor Server
---

**(注：昨天不小心把Reset按钮的type改成button，今天改回reset。)**

我们目前建立了3个组件，`Blog`、`Post`及`MyButton`，如果想让3个组件的字体颜色或是尺寸都一样，似乎得先在`MyButton`定义一个带有`[Parameter]` attribute的变量，`Post`调用时再填入值，`Post`同样定义变量，`Blog`调用时再填入，每个环节都要这样做。

前面讲过父组件想将数据传递到子组件内的就是靠`Parameter`，有没有类似`Parameter`不过可以一次传递到底下所有组件的方法呢？

这时候就要用到`CascadingValue`这个组件了，`cascading`的意思是`级联、喷泻`，可以将其理解为`由上而下注入`，前端用的CSS全名就是`Cascading Style Sheet`(层级式样式表)。

我们先在`BlogBase.razor.cs`加入一个Property `ColorStyle`(注：笔者之前文件取名有误，这篇改为BlogBase.razor.cs)，其值为`"color: goldenrod"`。

![](https://img1.dotnet9.com/2021/12/1801.png)

接着在`Blog.razor`用`<CascadingValue Value="ColorStyle">`将`<Post>`包住。

![](https://img1.dotnet9.com/2021/12/1802.png)

再于PostBase.razor.cs定义同名变量，不过多了`[CascadingParameter]`attribute。

![](https://img1.dotnet9.com/2021/12/1803.png)

然后只要在想要套用的地方填写该变量即可。

![](https://img1.dotnet9.com/2021/12/1804.png)

`MyButton`也是一样的用法。

![](https://img1.dotnet9.com/2021/12/1805.png)

虽然每个组件都还是要定义一个变量去承接最上层来的变量，不过可以看到`[CascadingParameter]`省去了在`PostBase.razor`调用`<MyButton>`必须填值的步骤。

那如果有两个值要这样层层传递呢？没错，就是包两层`<CascadingValue>`即可，但真的写了会发现，怎么会套用两个`color style`呢？

![](https://img1.dotnet9.com/2021/12/1806.png)

![](https://img1.dotnet9.com/2021/12/1807.png)

**[站长注：上下有差异，上面是站长调试结果，下面是原文截图]**

![](https://img1.dotnet9.com/2021/12/1808.png)

因为`<CascadingValue>`认识的是变量类型，如果类型不同可以方便套用，但如果有两个`string`，则子元素只会套用最靠近的`<CascadingValue>`，以图片为例就是`ColorStyle`。

为了解决这问题，可以使用里面的`Name`变量，如此一来组件之间就会认识这个`<CascadingValue>`的名字，如果子组件找不到该`Name`就会回传空白。

![](https://img1.dotnet9.com/2021/12/1809.png)

![](https://img1.dotnet9.com/2021/12/1810.png)

从上面的范例可以知道每次`<CascadingValue>`Blazor都会层层通知下层组件，但如果不希望每次都通知占用资源呢？可以用`IsFixed`，将其设为true，Blazor就不会去通知子组件了。

![](https://img1.dotnet9.com/2021/12/1811.png)

**引用：**

1. [Blazor cascading values and parameters](https://www.pragimtech.com/blog/blazor/blazor-cascading-values-parameters/)
2. [Blazor multiple cascading parameters](https://www.pragimtech.com/blog/blazor/blazor-multiple-cascading-parameters/)
3. [Blazor cascading values performance](https://www.pragimtech.com/blog/blazor/blazor-cascading-values-performance/)

**注：本文代码通过 .NET 6 + Visual Studio 2022重构，可点击原文链接与重构后代码比较学习，谢谢阅读，支持原作者**

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/12/2021-12-15_02.md)