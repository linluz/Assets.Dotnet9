---
title: Blazor Server 和 WebAssembly 应用程序入门指南
slug: getting-started-guide-for-blazor-server-and-webassembly-applications
description: 如果您一直紧跟 .NET 世界的最新发展趋势，那么现在您一定听说过 Blazor。
date: 2021-12-09 21:25:22
copyright: Reprint
author: 技术译民
originaltitle: Blazor Server 和 WebAssembly 应用程序入门指南
originallink: https://www.cnblogs.com/ittranslator/p/a-beginners-guide-to-blazor-server-and-webassembly-applications.html
draft: False
cover: https://img1.dotnet9.com/2021/12/cover_04.png
categories: Blazor
tags: Blazor
---

>翻译自 Waqas Anwar 2021年3月12日的文章 [《A Beginner’s Guide To Blazor Server and WebAssembly Applications》](https://www.ezzylearning.net/tutorial/a-beginners-guide-to-blazor-server-and-webassembly-applications)

![A-Beginner-Guide-To-Blazor-Server-and-WebAssembly-Applications](https://img1.dotnet9.com/2021/12/cover_04.png)

如果您一直紧跟 .NET 世界的最新发展趋势，那么现在您一定听说过 Blazor。目前在 .NET 社区中有很多关于 Blazor 的宣传，这种宣传最常见的原因是它引入了一些大多数 .NET 开发人员十几年来一直梦寐以求的东西，即：既可以在服务端又可以在浏览器中运行 C# 的能力。Blazor 允许我们使用 HTML、CSS 和 C#（而不是 JavaScript）来构建交互式 Web 应用程序。在本教程中，我将介绍 Blazor 的基本概念，并概述可用于 Blazor 的不同的托管模型。我还将介绍每种托管模型的优缺点，以便您可以为下一个 Blazor 项目托管模型做出最佳的决定。

## Blazor 是什么？

Blazor 是一个免费、开源的单页应用程序（SPA）开发框架，使开发人员能够在服务端和客户端上使用 C# 构建交互式 Web 应用程序。Blazor 不需要在客户端上安装任何插件来在浏览器中执行 C#/.NET 代码。它使用 WebAssembly 执行 .NET 代码，WebAssembly 是所有主流浏览器都支持的 Web 标准。Blazor 还可以在服务端运行 .NET 代码并构建 UI，然后通过 SignalR 连接仅将更新的 DOM 传输到客户端。

![BLAZOR](https://img1.dotnet9.com/2021/12/0401.png)

## WebAssembly 是什么？

WebAssembly（有时简写为 Wasm）是一种可移植的二进制格式（低级指令集），被设计用于在任何能够解释这些指令的主机上运行。WebAssembly 的主要目标是允许开发人员构建高性能的 Web 应用程序，但其格式也被设计为可执行于和集成到其他环境中。WebAssembly 目前受到了所有主流浏览器的支持，比如 Chrome、Android 版 Chrome、Edge、Firefox、Safari、Opera 等。

![WebAssembly](https://img1.dotnet9.com/2021/12/0402.png)

## Blazor 托管模型

Blazor 组件模型是 Blazor 的核心，它的设计方式使计算 UI 更改和呈现 UI 彼此分离。 这就是为什么无论您使用何种方式渲染您的应用程序，基本的组件模型都保持不变的原因。 在撰写本文时，有四种渲染/托管模型可用，它们都处于不同的开发阶段。

1. Blazor Server
2. Blazor WebAssembly
3. Blazor Electron
4. Mobile Blazor Bindings

Blazor Electron 和 Mobile Blazor Bindings 目前处于实验阶段，Microsoft 尚未承诺发布这些托管模型，因此我不会在本文中讨论它们。

## Blazor Server App 是什么？

Blazor Server 应用程序在服务器上运行，可享受完整的 .NET Core 运行时支持。所有处理都在服务器上完成，UI/DOM 更改通过 SignalR 连接回传给客户端。这种双向 SignalR 连接是在用户第一次从浏览器中加载应用程序时建立的。 由于 .NET 代码已经在服务器上运行，因此您无需为前端创建 API。您可以直接访问服务、数据库等，并在传统的服务端技术上做任何您想做的事情。

![Blazor-Server-Apps](https://img1.dotnet9.com/2021/12/0403.png)

### §何时使用 Blazor Server

1. 当您想在完整的 .NET Core 运行时上运行应用程序时
2. 当您想要保持应用程序的初始下载大小非常小时
3. 当您想保持应用启动时间非常快时
4. 当您想把应用程序的代码保留在服务器上，而不希望它被下载到客户端时
5. 当您想要一个快速的应用开发周期，而现有的 .NET 开发人员几乎不需要学习曲线时
6. 当您想让您的应用对搜索引擎友好时
7. 当您希望应用程序在旧的浏览器上运行，而不依赖于 WebAssembly 时
8. 当您想在 Visual Studio 中像调试普通 .NET 应用程序一样调试 .NET 代码时
9. 当您想要构建内部网或低需求的面向公众的应用程序时

### §何时不要使用 Blazor Server

1. 当您的应用程序在高延迟环境中运行时
2. 当您希望您的应用程序离线工作，而不需要一个与服务器的固定 SignalR 连接时
3. 当您不想增加服务器资源以响应大量连接的 SignalR 客户端时

## Blazor WebAssembly App 是什么？

这种托管模型是现代流行的 SPA 框架（比如 Angular、Vue 和 React）的直接竞争对手，也是大多数开发人员有兴趣学习 Blazor 的主要原因。它允许开发人员使用 C# 取代 JavaScript 编写所有前端 UI 逻辑。在这种托管模型中，第一次请求时，会将应用程序的 DLL 及其所有依赖项和小尺寸的 Mono .NET 运行时下载到客户端。然后，客户端中的 Mono 运行时就会加载并执行应用程序代码。Blazor WebAssembly 程序可以用 C、C# 等其他语言编写，然后编译成 WebAssembly 字节码。

![Blazor-WebAssembly-Apps](https://img1.dotnet9.com/2021/12/0404.png)

### §何时使用 Blazor WebAssembly

1. 当您想要将整个应用程序编译为静态文件，并将它们提供给客户端，而不需要服务器上的 .NET 运行时时。这意味着您的后端可以用 PHP、Node 或 Rails 编写，并服务于用 Blazor 编写的前端应用程序。
2. 当您想要构建可以在客户端脱机运行而无需持续连接到服务端的应用程序时。
3. 当您想要将处理转移到客户端并减少服务端的负载时。
4. 当您想在客户端和服务端之间共享代码和库时。

### §何时不要使用 Blazor WebAssembly

1. 当由于下载到客户端的文件/DLL 太多，而您无法在有效负载上妥协时。
2. 当您无法在缓慢的启动时间上（特别是在网络连接不佳的情况下）妥协时。
3. 当您无法妥协于应用程序必须运行在具有完整的安全限制的浏览器沙箱环境中时。

为了更好地理解 Blazor 托管模型，让我们在 Visual Studio 2019 中分别创建一个 Blazor Server 和 Blazor WebAssembly 应用程序。

## 在 Visual Studio 2019 中创建 Blazor Server 应用

打开 Visual Studio 2019 并点击创建新项目。从可用的模板列表中选择 Blazor App 模板并点击下一步。

![Create-New-Blazor-App-Project-in-Visual-Studio-2019](https://img1.dotnet9.com/2021/12/0405.png)

指定项目名称（比如 BlazorServerApp）并点击下一步。您将看到下面的对话框，询问您选择要创建的 Blazor 应用程序的类型。我们要创建 Blazor Server 应用，所以请选择 Blazor Server App 并点击 创建 按钮。

![Blazor-Server-App-in-Visual-Studio-2019](https://img1.dotnet9.com/2021/12/0406.png)

Visual Studio 将为我们创建一个 Blazor Server 应用程序，其中在解决方案资源管理器中包含以下文件夹和文件。

![Blazor-Server-App-in-Solution-Explorer](https://img1.dotnet9.com/2021/12/0407.png)

让我们来讨论一下 Blazor Server App 中可用的一些重要文件和文件夹。

### §Program.cs

这个文件包含 Main 方法，它是项目的入口点。Main 方法中调用 CreateHostBuilder 方法，为我们配置默认的 ASP.NET Core 宿主。

```C#
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }
 
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<startup>();
            });
}
```

### §Startup.cs

它与我们在标准 ASP.NET Core 项目中使用的文件相同。需要重点注意的一点是 `ConfigureServices` 方法中调用了 `AddServerSideBlazor`，该方法添加 Blazor Server App 相关的服务。

```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddRazorPages();
    services.AddServerSideBlazor();
    services.AddSingleton<weatherforecastservice>();
}
```

在此文件的 `Configure` 方法中我们还有以下两行重要的代码。`MapBlazorHub` 方法配置 Blazor Server App 所需的 SignalR Hub Endpoints。`MapFallbackToPage` 方法会将所有未与任何控制器、razor 页面等匹配的请求映射到 **_Host** 页面，这将允许所有动态内容请求路由到 SPA 框架，而不是抛出 404 Not Found。

```C#
app.UseEndpoints(endpoints =>
{
    endpoints.MapBlazorHub();
    endpoints.MapFallbackToPage("/_Host");
});
```

### §_Host.cshtml

这是应用程序的根页面，每个 Razor 组件/页面都将在此 host 页面中呈现。它具有基本的 HTML 元素，例如 html、head 和 body，以及一些特殊元素。请注意，Blazor 是一个基于组件的框架，Blazor 中的每一内容都是一个组件。`<component>` 指定了我们想让应用程序根组件呈现的位置。

```html
<component type="typeof(App)" render-mode="ServerPrerendered">
```

该文件还在末尾注入了 **blazor.server.js** 文件，此 JavaScript 文件包含设置 SignalR 连接到服务端的代码。此连接在浏览器加载应用程序后立即建立，然后被用于服务端和客户端浏览器之间的实时通信。如果您想了解有关 SignalR 的更多知识，请阅读我的文章 [Display Live Sports Updates using ASP.NET Core SignalR](https://www.ezzylearning.net/tutorial/display-live-sports-updates-using-asp-net-core-signalr)。

```html
<script src="_framework/blazor.server.js"></script>
```

### §App.razor

这是 Blazor App 的主要组件，其主要工作是拦截路由并呈现 **Found** 或 **NotFound** 组件。 如果找到与路由匹配的组件，则呈现 **Found** 组件，如果未找到匹配的组件，则呈现 **NotFound** 组件。

```html
<Router AppAssembly="@typeof(Program).Assembly" PreferExactMatches="@true">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
    </Found>
    <NotFound>
        <LayoutView Layout="@typeof(MainLayout)">
            <p>Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>
```

### §MainLayout.cshtml

MainLayout 文件包含应用程序的主布局，其标记可以被多个 Razor 组件共享。这个布局组件通常包含应用程序的常见 UI 元素，例如页眉、菜单、页脚、侧边栏等。为我们生成的默认 MainLayout 有一个侧边栏，用来渲染 **NavMenu** 组件，它还使用 Razor 语法 **@Body** 来指定其他组件的内容将在布局标记中呈现的位置。

```html
@inherits LayoutComponentBase
 
<div class="page">
    <div class="sidebar">
        <NavMenu />
    </div>
 
    <div class="main">
        <div class="top-row px-4">
            <a href="https://docs.microsoft.com/aspnet/" target="_blank">About</a>
        </div>
 
        <div class="content px-4">
            @Body
        </div>
    </div>
</div>
```

### §wwwroot 文件夹

该文件夹包含静态文件，例如图片、字体、图标、CSS 和 JavaScript 文件等。

### §Pages 和 Shared 文件夹

该文件夹包含我们之前讨论过的 _Host.cshtml 文件以及一些 Razor 组件。Blazor 应用程序是具有 .razor 扩展名的 Razor 组件的集合。其中一些组件称为可路由组件，因为可以使用其路由访问它们。例如，当我们导航到应用程序根 URL 时，将呈现下面的 **Index.razor** 组件。该 URL 是使用 **Index.razor** 组件顶部的 **@page** 指令指定的。

### §Index.razor

```html
@page "/"
 
<h1>Hello, world!</h1>
 
Welcome to your new app.
 
<SurveyPrompt Title="How is Blazor working for you?" />
```

请注意，上面的页面还使用了一个子组件 **SurveyPrompt**，之所以称它为子组件，是因为它没有 **@page** 指令，它可以被嵌入到其他组件中。

Pages 文件夹中还有一些其他的 razor 组件，这些组件都可以使用文件顶部指定的路径进行访问。例如，当我们导航到 **/counter** 路径时，**Counter** 组件将呈现。类似地，**FetchData** 组件将使用 **/fetchdata** 路径呈现。

Razor Server 应用程序还有一个包含共享组件的 Shared 文件夹。这些组件可以被整个应用程序中的任何组件使用，就像我们上面看到的 SurveyPrompt 组件一样。Shared 文件夹中另一个有趣的共享组件是 NavMenu 组件，它呈现 Blazor Server 应用程序的顶部导航栏。

### §_Imports.razor

该文件类似于我们在 ASP.NET MVC Web 应用程序中的 _ViewImports.cshtml 文件，它包含我们可以在不同 razor 组件中使用的命名空间列表。在 _Imports.razor 文件中声明所有这些命名空间的好处是我们不需要在每个 razor 组件中重复引入它们。

```html
@using System.Net.Http
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.Forms
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.Web
@using Microsoft.AspNetCore.Components.Web.Virtualization
@using Microsoft.JSInterop 
```

现在是时候运行我们的 Blazor Server 应用程序并在浏览器中查看它的运行情况了。在 Visual Studio 中按 `F5`，您将看到一个漂亮的默认 Blazor Server 应用程序。试试从侧边栏导航到不同的页面，并尝试在 Counter 页面上使用计数器，您会注意到没有页面刷新或回传到服务器。一切都像经典的 SPA 那样流畅和快速，浏览器和服务端的所有通信都是使用 SignalR 连接进行的。

![Default-Blazor-Server-App-Running-in-Browser](https://img1.dotnet9.com/2021/12/0408.png)

您也可以打开浏览器开发者工具，您会注意到所有标准的 CSS 和 JavaScript 文件（包括 blazor.server.js 文件）都下载到了客户端，并通过 Web Sockets 建立了一个 SignalR 连接。

![Blazor-Server-App-Files-in-Browser-Developer-Tools](https://img1.dotnet9.com/2021/12/0409.png)

## 在 Visual Studio 2019 中 创建 Blazor WebAssembly 应用

我们已经了解了 Blazor Server App 的基础知识，并在浏览器中看到了它的运行情况。现在让我们创建一个 Blazor WebAssembly App，以便我们可以理解它们的不同之处。按照我们上面提到的相同步骤，并使用 **Blazor App** 模板在 Visual Studio 中创建一个新的 Blazor 应用程序。当您被询问选择 Blazor App 的类型时，这次需要选择 **Blazor WebAssembly App**。

![Create-Blazor-WebAssembly-App-in-Visual-Studio-2019](https://img1.dotnet9.com/2021/12/0410.png)

Visual Studio 将为我们创建一个 Blazor WebAssembly 应用程序，其中在解决方案资源管理器中包含以下文件夹和文件。

![Blazor-Client-App-in-Solution-Explorer](https://img1.dotnet9.com/2021/12/0411.png)

您可以轻松发现这两种类型的应用程序之间的一些差异。例如，在 Blazor WebAssembly 应用程序中没有以下文件：

1. _Host.cshtml
2. Error.cshtml
3. Startup.cs
4. appsettings.json

### §index.html

在 Blazor WebAssembly 应用程序中，我们会在 **wwwroot** 文件夹中有一个 **index.html** 文件，作为主页面， 该文件在末尾注入了 **blazor.webassembly.js** 文件，此文件由框架提供以处理下载 .NET 运行时、Blazor 应用程序及其所有依赖项。它还包含为了运行应用而初始化运行时的代码。

### §Program.cs

在 Blazor WebAssembly 应用程序中，应用程序的根组件在 Program.cs 文件中的 Main 方法中指定。应用程序的根组件是 **App.razor**，你可以看到它是如何被添加到 RootComponents 集合中的。

```C#
public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebAssemblyHostBuilder.CreateDefault(args);
        builder.RootComponents.Add<app>("#app");
 
        builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });
 
        await builder.Build().RunAsync();
    }
}
```

在 Visual Studio 中按 `F5`，您将看到一个相似的 Blazor WebAssembly 应用程序。尝试从侧边栏导航到不同的页面，并尝试像之前在 Blazor Server App 中所做的那样在 Counter 页面上使用计数器。 一切看起来感觉一模一样，也没有服务器端回传。

![Default-Blazor-Server-App-Running-in-Browser](https://img1.dotnet9.com/2021/12/0412.png)

正如我们已经知道的那样，Blazor WebAssembly 应用程序会在客户端下载应用程序及其所有依赖项，因此如果您打开浏览器开发者工具，会看到客户端下载了大量 DLL（只会在第一次浏览时下载）。

![Blazor-Client-App-Files-in-Browser-Developer-Tools](https://img1.dotnet9.com/2021/12/0413.png)

以上所有文件只会在第一次请求时下载，然后它们被缓存在浏览器中。如果您再次刷新页面，将会看到这一次下载的文件很少。

![Blazor-Client-App-Files-in-Browser-Developer-Tools-Second-Request](https://img1.dotnet9.com/2021/12/0414.png)

## 总结

在本文中，我试图为您介绍 Blazor SPA 框架的基本概念，我们看到了两个 Blazor 应用程序使用两种不同的托管模型进行托管。因为 Blazor 框架严重依赖于 razor 组件，所以两个项目中的大部分代码和文件都是相同的。这些组件是 Blazor 应用程序的构建块，无论使用哪种托管模型，我们都可以以相似的方式构建这些组件。如果您喜欢本文，请分享它并传播知识。

>作者： Waqas Anwar
>
>翻译： [技术译站](https://ittranslator.cn/)
>
>链接： [英文原文](https://www.ezzylearning.net/tutorial/a-beginners-guide-to-blazor-server-and-webassembly-applications)

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/12/2021-12-09_01.md)