---
title: AOT和单文件发布对程序性能的影响
slug: Influence-of-AOT-and-single-file-release-on-program-performance
description: 以前的.NET框架原生并不支持最终编译结果的单文件发布（需要依赖第三方工具）
date: 2022-04-20 07:31:25
copyright: Reprint
author: InCerry
originaltitle: AOT和单文件发布对程序性能的影响
originallink: https://www.cnblogs.com/InCerry/p/Single-File-And-AOT-Publish.html
draft: False
cover: https://img1.dotnet9.com/2022/04/cover_24.jpg
categories: .NET相关
tags: C#,AOT
---

## 1. 前言

这里先和大家介绍一下.NET一些发布的历史，以前的.NET框架原生并不支持最终编译结果的单文件发布（需要依赖第三方工具），我这里新建了一个简单的ASP.NET Core项目，发布以后的目录就会像下图这样，里面包含很多`*.dll`文件和其它各类的文件。

![](https://img1.dotnet9.com/2022/04/2401.png)

在.NET Core 2.1时代，引入了单文件发布的功能，只需要在发布命令上，增加`-p:PublishSingleFile=true`参数就可以使用，从这以后就无需发布的文件夹就再也没有那么多的文件，只有一个`*.exe`文件和对应的配置文件和用于调试`*.pdb`的文件，如下所示：

![](https://img1.dotnet9.com/2022/04/2402.png)

不过此时的.NET还是需要安装一个大小为50~130MB左右的.NET Runtime才能运行，这个其实不利于在客户端场景下程序的分发，大家应该能回忆起在安装一些软件之前，必须安装.NET Framework的场景。

![](https://img1.dotnet9.com/2022/04/2403.png)

在单文件发布推出的同时，也可以通过`--self-contained true`的参数，将运行时也包含在发布文件内，这样的话就无需在目标机器上再安装.NET Runtime。不过由于它自带运行时，整个发布文件夹的大小就变得很大了，可以说比安装.NET Runtime还要大一些（足足82.4MB）。

![](https://img1.dotnet9.com/2022/04/2404.png)

程序本质上也就是文件，我们也可以通过压缩程序的方式，让它的大小变小，只需要加上`-p:EnableCompressionInSingleFile=true`参数。就可以将80MB的程序压缩至44MB左右。

![](https://img1.dotnet9.com/2022/04/2405.png)

单文件发布体积大的原因就是包括了所有运行可能用到的依赖，不过有很多依赖是我们程序中用不到的，所以发布的时候可以加`-p:PublishTrimmed=true`参数，发布的时候移除掉没有使用的依赖，这样体积就可以降低很多（从44MB到35MB）。

![](https://img1.dotnet9.com/2022/04/2406.png)

当然，移除没有使用的依赖和压缩是可以同时使用的，这样发布以后，体积就可以变得更小了，只需要20MB左右。

![](https://img1.dotnet9.com/2022/04/2407.png)

此时.NET运行还是需要自带运行时，在运行.NET程序的时候需要JIT来参与，这样的话在应用启动时需要一定的时间让JIT将MSIL编译到对应平台机器码，随后.NET推出了预览版的`Native-AOT`，可以在编译时直接将代码编译成对应平台的机器码，以加快启动速度；另外由于不需要自带运行时，它整个的体积大小也变得很小。

![](https://img1.dotnet9.com/2022/04/2408.png)

用于调试的`pdb`文件就会变得很大，不过真实发布的话也用不到这个文件，可以舍弃。AOT以后的大小也就20MB左右。不过AOT也不是银弹，由于没有了JIT，很多编译时优化就不能做了，Java的GraalVm发布的时候就有一张五边形图，充分的说明了JIT和AOT之间的取舍。

![](https://img1.dotnet9.com/2022/04/2409.png)

AOT拥有更快的启动速度、更低的内存占用和更小的程序体积；当然它的吞吐量和最大延时表现的就没那么好（另外也会失去很多动态的特性，降低一些编程效率）。

心中会有一个疑问，这样的发布方式会对程序的性能有影响嘛？都说AOT会让程序启动速度变快，那么会变快多少呢？

## 2. 评测结果

我决定花点时间来研究一下，周末带着上面的问题我设计了一组测试，当然时间仓促有很多不严谨的地方，可以说就图一乐，望大家指出和海涵。一共设计了12个组，主要是对比单文件发布、AOT发布和普通发布的区别；另外我也加入了PGO、TC、OSR和OSA等JIT参数，来看看不同JIT参数的影响。

>PGO：PGO 即 Profile Guided Optimization（配置引导优化），通过收集运行时信息来指导 JIT 如何优化代码，相比以前没有 PGO 时可以做更多以前难以完成的优化。可以参考hez大佬的[博客](https://www.cnblogs.com/hez2010/p/optimize-using-pgo.html)，还有一些[链接1](https://gist.github.com/EgorBo/dc181796683da3d905a5295bfd3dd95b#:~:text=Dynamic%20PGO%20in.NET%206.0%20Dynamic%20PGO%20%28Profile-guided%20optimization%29,hot%20methods%20to%20make%20them%20even%20more%20efficient.)、[链接2](https://devblogs.microsoft.com/dotnet/conversation-about-pgo/)、[链接3](https://blog.csdn.net/u010285974/article/details/86090417).

>TC：TC 即 Tiered Compilation（分层编译），是一种运行时优化代码的技术，每个C#函数都会由JIT编译成目标平台的机器码，为了让方法能快点运行，JIT一般会很粗犷(并不是最优，生成代码效率比较低)的编译，所以JIT就引入了TC，当某一个方法频繁被调用时，JIT就会为它编译一份更优的代码，这样下一次方法被调用时，它执行的会更有效率。想了解更多关于.NET分层编译可以戳这个[链接](https://devblogs.microsoft.com/dotnet/tiered-compilation-preview-in-net-core-2-1/)。

>OSR：OSR 即 On-Stack Replacement（栈上替换），OSR是一种在运行时替换正在运行的函数/方法的栈帧的技术。这个是为了分层编译引入的，因为有时候我们运行的方法是一个`while(ture)`这种死循环方法，分层编译找不到时机能把低优化的代码替换成高优化的代码，所以引入了栈上替换，在方法运行中就可以替换成更优的方法。[链接1](https://github.com/dotnet/runtime/issues/33658)、[链接2](https://www.zhihu.com/question/45910849/answer/100636125)。

>OSR：OSA 即 Object Stack Allocation （对象栈上分配），在.NET中的引用对象默认是分配在堆上的，回收时需要垃圾回收器介入，而且分配对象时必须初始化内存（全部初始化为0），如果对象的生命周期可控，那么可以将它分配在栈上。这样做的好处就是能降低GC压力（方法栈结束，对象自动释放了），提升性能（可以进行标量替换，访问更快）。[链接1](https://github.com/dotnet/runtime/blob/main/docs/design/coreclr/jit/object-stack-allocation.md)。

每个组的命名和参数如下所示。

| 项目 | 备注 |
| ---- | ---- |
| Normal | 正常发布，对照组 |
| Normal-WksGC | 正常方式，使用WorkStationGC |
| Normal_PGO | 正常发布，使用PGO |
| Normal_PGO_OSR | 正常发布，使用OSR |
| Normal_PGO_OSR_OSA | 正常发布，使用PGO+OSR+OSA |
| SingleFilePublish | 普通单文件发布 |
| SingleFilePublish-SelfContained | 包含运行时单文件发布 |
| SingleFilePublish-SelfContained-Trim | 包含运行时单文件发布+剪裁程序集 |
| SingleFilePublish-SelfContained-Compress | 包含运行时单文件发布+压缩程序集 |
| SingleFilePublish-SelfContained-Trim-Compress | 包含运行时单文件发布+剪裁+压缩程序集 |
| AOT-Size | AOT编译，使用Size模式 |
| AOT-Speed | AOT编译，使用Speed模式 |

下方的小标题是评测项的方式和评测的结果，每个项我们都会跑5次，最后取平均值。

### 2.1 发布相关

在本节中，Normal那几项编译参数都是一样的，所以结果几乎没有差别，无需过多关注，忽略就好。

#### 2.1.1 发布耗时

发布耗时这个参数，是记录了`dotnet publish`的耗时，其中会清理`/bin`、`/obj`等文件夹，避免缓存带来的影响。

![](https://img1.dotnet9.com/2022/04/2410.png)

可以看到单文件发布和AOT发布还是比较吃性能的，特别是AOT场景下简单的ASPNET Core项目的发布时间就到了接近30秒和一些Rust、C++项目编译速度有的一拼了，要是更大的项目估计会更长。不过正常发布还是很快的，不会一两秒内都能完成。

#### 2.1.2 目录大小

目录大小是直接统计发布以后的目录所占用的硬盘空间，注意：Normal发布都计算了67.5MB的.NET Runtime占用的空间。

![](https://img1.dotnet9.com/2022/04/2411.png)

为什么AOT的目录大小会这么大呢？主要就是上文中提到的用于调试程序的`pdb`文件变的很大，这是因为AOT以后程序本身缺失很多用于调试的数据，只能存放在`pdb`文件中，不过这个对于使用没有什么影响，发布时也可以通过`-p:DebugType=false`和`-p:DebugSymbols=false`参数让它不生成`pdb`文件。

#### 2.1.3 程序大小

程序大小统计只发布文件中需要运行程序的大小，这个是和分发项目息息相关的，越小的程序体积，就越容易分发。注意：Normal发布都计算了67.5MB的.NET Runtime占用的空间。

![](https://img1.dotnet9.com/2022/04/2412.png)

如果目标平台已经预装了.NET Runtime，其实正常发布的效率是最高的，只有一百多KB的大小；次之就是单文件发布+自包含运行时+裁剪+压缩，大小只有20来MB，也比较利于分发。AOT的表现也同样亮眼。

### 2.2 程序运行相关

程序运行相关一共有三个指标，分别为启动耗时、应用启动耗时和内存占用，这里没有设置CPU相关的指标，是因为启动程序CPU基本都是0没有太大的参考意义。下方流程图展示了这几个指标的采集时间。

![启动配图](https://img1.dotnet9.com/2022/04/2413.png)

#### 2.2.1 启动耗时

程序的启动耗时结果如下所示。

![](https://img1.dotnet9.com/2022/04/2414.png)

我们可以看到两个极值，最大的单文件+自包含运行时+压缩启动耗时到170ms，因为没有剪裁程序集，需要解压缩的依赖很大，所以启动耗时会比较长一点。最小的AOT-Speed模式只需要16.8ms就能启动程序，看来没有了JIT编译和程序集加载的过程，果然快很多。

#### 2.2.2 应用启动耗时

![](https://img1.dotnet9.com/2022/04/2415.png)

应用启动耗时和程序启动耗时排列基本一致，像单文件+自包含运行时+压缩启动耗时需要0.5s+才能启动程序，而AOT模式只需要70ms，中间差了七八倍。不过正常发布启动速度也很快，只需要200ms不到的时间。

#### 2.2.3 内存占用

![](https://img1.dotnet9.com/2022/04/2416.png)

内存占用各个方式差别不大，但是也提醒到了我们，如果想让内存占用小一些，那么可以使用WorkstationGC模式。引入动态PGO之类的JIT增强特性以后，相应的会多占用一些内存。

### 2.3 性能压测

机器配置：

>CPU：I7 8750H 关闭超线程
>
>RAM：48GB
>
>Client：设置CPU亲和性，绑定3个核心
>
>Server：设置CPU亲和性，绑定2个核心

由于笔者机器配置有限，没有做`Client`和`Server`的环境隔离，只做了简单的CPU绑核，所以的出来的数据仅供参考。

#### 2.3.1 压测QPS

![](https://img1.dotnet9.com/2022/04/2417.png)

可以看到其实各个方式差别不是很大，都取得了`4.7Wqps`以上的成绩，最大和最小在4%以内。由于这是IO密集型任务，JIT、PGO的优势没有体现出来，后面可以试试一些计算密集型的任务，或者直接看hez的博客，上文介绍PGO中有链接。

#### 2.3.2 单次请求耗时

下图中在条形图内较大的是`单次请求耗时(MAX)`，在条形图外的`0.x`的数据是`单次请求耗时(AVG)`。单位是`ms`.

![](https://img1.dotnet9.com/2022/04/2418.png)

我们发现平均耗时基本在`0.3ms`左右，AOT和单文件+自包含运行时+剪裁+压缩的表现很亮眼，只有`370ms`左右。

#### 2.3.3 压测内存占用

下图中深色代表`内存占用(MAX)`而浅色代表`内存占用(AVG)`，单位是`MB`.

![](https://img1.dotnet9.com/2022/04/2419.png)

可以看到除了AOT以外的方式，内存占用是大差不差的，`4.7Wqps`下只需要`25MB`左右的内存其实很不错了，近似的数字可以理解为误差；另外开启了JIT特性以后，就需要占用更多的内存。AOT的话内存占用就比较多了，可能GC算法在AOT环境下的优化还不够。

#### 2.3.4 压测CPU占用

下图中深色代表`CPU占用(MAX)`而浅色代表`CPU占用(AVG)`。单位为`百分比`；1个CPU核心是`100%`，如果占用5个CPU核心那么就是`500%`。

![](https://img1.dotnet9.com/2022/04/2420.png)

基本上都没有啥区别，但是AOT方式占用率就小了很多，毕竟没有了JIT这个步骤。

## 3. 总结

这个结论也就是图一乐，毕竟目前AOT还没有正式发布(已经合并主分支.NET7会正式发布)，还有很多值得优化的地方。另外像OSR、OSA这些特性也还没有完全定下来，下面是一些和对照组比较的百分比数据，原始数据和测试代码见[Github](https://github.com/InCerryGit/BlogCode-Single-File-And-AOT-Publish)。后续.NET7正式发布了，再跑一下试试。

![](https://img1.dotnet9.com/2022/04/2421.jpg)

![](https://img1.dotnet9.com/2022/04/2422.jpg)

回答开始提到的问题，总得来说AOT对缩小软件大小，提升应用启动速度有着很大的作用，但是目前需要很长的发布时间和占用更多的内存。

另外PGO等一些JIT特性需要比正常情况下占用更多的内存，其性能的优势在这个IO密集的场景没有很好的表现出来。

最后在多说几句，我一直觉得C#是一个很好的语言，.NET是一个很好的平台。从2002年一路走来，今年是.NET的第20个年头，各种新特性相继加入，性能也已经站在了第一梯队，希望以后能有更多的发展吧。

>PS：在前几天更新的Benchmarks Game数据里面，C# .NET已经是带JIT语言里面跑的最快的了，仅次于C、C++、Rust等编译型语言，详情可见[链接1](https://github.com/GoodManWEN/Programming-Language-Benchmarks-Visualization)、[链接2](https://benchmarksgame-team.pages.debian.net/benchmarksgame/index.html)。

![](https://img1.dotnet9.com/2022/04/2423.png)

>原文作者：InCerry
>
>原文标题：单文件发布对程序性能的影响
>
>原文链接：https://www.cnblogs.com/InCerry/p/Single-File-And-AOT-Publish.html