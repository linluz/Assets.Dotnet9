---
title: (9/30)大家一起学Blazor：Parameters
slug: 9-of-30-let-us-learn-blazor-together-parameters
description: 假如我们想增加的按钮用来清除`form`（表单）的数据，最快的方式是增加一个`type=”reset”`的按钮，这时候就用到两个按钮了，可以用到Blazor的核心概念：组件封装。
date: 2021-12-14 22:18:54
copyright: Reprint
author: StrayaWorker
originaltitle: (9/30)大家一起学Blazor：Parameters
originallink: https://ithelp.ithome.com.tw/articles/10261943
draft: False
cover: https://img1.dotnet9.com/2021/12/cover_05.png
albums: 学Blazor
categories: Blazor
tags: Blazor Server
---

假如我们想增加的按钮用来清除`form`（表单）的数据，最快的方式是增加一个`type=”reset”`的按钮，这时候就用到两个按钮了，可以用到Blazor的核心概念：组件封装。

我们先在Shared文件夹新增一个Razor组件名为`MyButton`，定义3个变量分别代表按钮类型、按钮样式、按钮名称，注意要在属性上面加上`[Parameter]`，这是告诉Blazor这些属性的值来自调用这个组件的父组件。

![](https://img1.dotnet9.com/2021/12/1501.png)

接着移除标题的单向绑定，再将更新时间的div移到`<EditForm>`外面，因为reset按钮会清除整个form，我们不希望更新时间也被清除。最后在最下面使用两个`<MyButton>`组件，再输入我们要的按钮类型、按钮样式、按钮名称，这边Blazor会告诉你还有哪些Parameter没有被使用，如果有重复的Parameter也会有提示，所以不用担心。

![](https://img1.dotnet9.com/2021/12/1502.png)

按下Reset按钮，可以看到标题跟内容都被清除了。

![](https://img1.dotnet9.com/2021/12/1503.gif)

要注意的是Parameter不支援混合razor及html字符串的写法，如果想这么做建议用`field`、`property`或是`string`方法回调组合好的字符串。

![](https://img1.dotnet9.com/2021/12/1504.png)

![](https://img1.dotnet9.com/2021/12/1505.png)

Parameter还有一点要避免，当父组件传值到子组件时，父组件若有`StateHasChanged`相关动作，会将父组件及子组件重新渲染，除非在子组件用一个变量保存父组件传来的值。

上面的文字说明似乎没看懂，我们来看微软给的例子，先建立一个扩展组件，里面做的事情很简单，只要点击`<div>`就会收缩或是展开内容。还有一个类型为`RenderFragment`的`ChildContent`，这是让使用该组件的外部组件决定内容模板。

![](https://img1.dotnet9.com/2021/12/1506.png)

接着在Post.razor调用两个`Expander`，第一个有用到`ChildContent`并放入`<p>`元素，最后看到一个button带有`@onclick="StateHasChanged"`的事件，模拟父组件的`StateHasChanged`事件。

![](https://img1.dotnet9.com/2021/12/1507.png)

打开网页可以看到两个`Expander`的`Expanded`皆为True

![](https://img1.dotnet9.com/2021/12/1508.png)

接着两个Expander都点一下，可以看到`Expanded`都变False了

![](https://img1.dotnet9.com/2021/12/1509.gif)

不过如果这时候点底下的button，会发现只有上面的Expanded变成True，这是为什么？

![](https://img1.dotnet9.com/2021/12/1510.gif)

原因就在于父组件的状态（state）跟`ChildContent`这个属性，因为父组件的state改变了会重新渲染（render）并将新数据传给第一个Child，所以`Expanded`被重置为true，第二个Child因为没有接收`ChildContent`所以不会重新渲染。

为了避免这问题，可以定义一个私有字段（field）名为`_expanded`，当组件初始化时保存父组件传来的`Expanded`，之后的逻辑都根据`_expanded`处理。

![](https://img1.dotnet9.com/2021/12/1511.png)

**引用：**

1. [Component parameters](https://docs.microsoft.com/en-us/aspnet/core/blazor/components/?view=aspnetcore-5.0#component-parameters-1)
2. [ComponentBase.StateHasChanged Method](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.components.componentbase.statehaschanged?view=aspnetcore-5.0)

**注：本文代码通过 .NET 6 + Visual Studio 2022重构，可点击原文链接与重构后代码比较学习，谢谢阅读，支持原作者**

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/12/2021-12-14_01.md)