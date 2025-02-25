---
title: 预览 C# 10 的新东西
slug: preview-what-is-new-in-csharp-10
description: 学习永不止步
date: 2021-06-01 22:19:09
copyright: Reprint
author: Ken Bonny&Rwing
originaltitle: 预览 C# 10 的新东西
originallink: https://www.cnblogs.com/Rwing/p/introducing-csharp-10.html
draft: False
cover: https://img1.dotnet9.com/2021/06/cover_05.png
categories: .NET相关
tags: C#
---

- 原文: [Introducing C# 10](https://kenbonny.net/introducing-csharp-10)
- 作者: [Ken Bonny](https://hashnode.com/@KenBonny)
- 翻译：[Rwing](https://home.cnblogs.com/u/Rwing/)
- 译文：[[翻译] 预览 C# 10 的新东西](https://www.cnblogs.com/Rwing/p/introducing-csharp-10.html)


本周早些时候(译注:原文发表于5月1日)，我关注了 [Mads Torgersen](https://twitter.com/MadsTorgersen) 在 [DotNet SouthWest](https://www.meetup.com/dotnetsouthwest/) 大会上的[演讲](https://www.youtube.com/channel/UCU0f_2rwIlvLC35GfGk_8kg)，他是微软的 C# 语言的首席设计师。他概述了 C# 10 即将包含的很酷的一些新东西。让我们来快速浏览一下。

>小小的免责声明，这些变化中的大部分已经基本完成。但是由于它仍在积极的开发中，我不能保证 C# 10 发布时所有东西都会完全如实。

## struct record

他谈到的第一件事是，目前 ``record`` 的实现是使用一个 ``class`` 作为基础对象的。现在还会有一个 ``record struct`` 的变体可用，所以基础类型可以是一个值类型。区别在于，普通的 ``record`` 在函数之间传递的是引用，而 ``record struct`` 是其值的拷贝。 ``record struct`` 也会支持 ``with`` 运算符。

同时，还允许向 record添加运算符，两种 record 类型都可以。

```C#
record Person(string Name, string Email)
{
  public static Person operator +(Person first, Person second)
  {
    // logic goes here
  }
}
```

## required

C# 团队关注的目标之一是使对象的初始化变得更容易。这就是为什么可以对 class、struct、record 或 struct record 的属性添加 required 标记 。它使得这些属性必须填写。这可以通过构造函数来完成，也可以通过对象初始化来完成。下面的两个类的定义是等价的。如果你添加了 required 关键字，那么就无法在不设置 Name 属性的情况下将Person 实例化。编译器会抛出错误，无法编译。

```C#
class Person
{
  public required string Name { get; set; }
  public DateTime DateOfBirth { get; set; }
}
```

```C#
class Person
{
  public Person(string name) => Name = name;

  public string Name { get; set; }
  public DateTime DateOfBirth { get; set; }
}
```

## field

为了进一步的改善属性，将允许完全摆脱 backing field 。新的关键字 field 将提供对上述字段的访问。对setter 和 init only 属性都可以使用。

```C#
class Person
{
  public string Name { get; init => field = value.Trim(); }
  public DateTime DateOfBirth { get; set => field = value.Date; }
}
```

## with

在下一个版本中还会有一些有趣的小改进。其实中一个是匿名类型也将支持 ``with`` 运算符。

```C#
var foo = new
{
  Name = "Foo",
  Email = "foo@mail.com"
};
var bar = foo with {Name = "Bar"};
```

## namespace

现在可以创建一个带有命名空间导入的文件，然后在任何地方都可以使用这个导入。例如，如果有一个很常用的命名空间，几乎在每个文件中都使用例如 ``Microsoft.Extensions.Logging.ILogger`` ，那么就可以在任何.cs文件（我建议在 Program.cs 或专门的 Imports.cs ）中添加一行 ``global using Microsoft.Extensions.Logging.ILogger``，之后这个命名空间将可以在整个项目中使用。注意，这不适用于整个解决方案! 没有人能够预测哪些地方需要导入，所以它们被分组到每个项目中。

```C#
// 译注：原文并没有提供代码示例，为了更好方便大家理解私自添加了一个演示
// Program.cs 文件
global using System;

// Sample.cs 文件
// 可以不用再using System;
Console.WriteLine("foo");
```

随后，还将对命名空间也会有一个优化。现在命名空间需要大括号 {} 来包起来代码，这就意味着所有代码至少要缩进一次。为了节省 tab（或四个空格）和屏幕空间，在文件的任何地方添加一个命名空间，将使所有代码都属于该命名空间。有研究表明绝大多数情况下，一个文件中所有的代码都属于同一个命名空间。使用这个方案后，文件大小随之减少，这对一个解决方案来说可能并不明显（即使它包含成千上万的文件），但在GitHub/GitLab/BitBucket/... 的规模上，我认为这将为他们节省一些空间。如果有人仍想在一个文件中包含多个命名空间，使用大括号的选项仍然可用。

```C#
// 传统的方式 LegacyNamespace.cs
namespace LegacyNamespace
{
  class Foo
  {
    // legacy code goes here
  }
}

// 简化的方式 SimplifiedNamespace.cs
namespace SimplifiedNamespace;
class Bar
{
  // awesome code goes here
}
```

## lambda

对 lambda 语句也会有一些很酷的更新。编译器将对推断 lambda 签名提供更好的支持，而且还可以添加特性。话可以显式指定返回类型，以帮助编译器理解 lambda。

```C#
var f = Console.WriteLine;
var f = x => x; // 推断返回类型
var f = (string x) => x; // 推断签名
var f = [NotNull] x => x; // 添加特性
var f = [NotNull] (int x) => x;
var f = [return: NotNull] static x => x; // 添加特性
var f = T () => default; // 显式返回类型
var f = ref int (ref int x) => ref x; // 使用 ref 
var f = int (x) => x; // 显式指定隐式输入的返回类型
var f = static void (_) => Console.Write("Help");
```

>感谢 Schooley 提出了一个不那么容易混淆的特性例子

## interface

最后，将有可能在接口上指定静态方法和属性。我知道这将是一个有争议的话题，就像给接口添加默认实现一样。我不喜欢它。然而，这可能非常有趣。想象一下，你可以指定一个接口的默认值或指定创建方法。

```C#
interface IFoo
{
  static IFoo Empty { get; }
  static operator +(IFoo first, IFoo second);
}
class Foo : IFoo
{
  public static IFoo Empty => new Foo();
  public static operator +(IFoo first, IFoo second) => /* do calculation here */;
}
```

就个人而言，我喜欢这些变化。我最喜欢的是对命名空间的改变和对接口的改进。总之，未来是光明的 C# 的。嗯嗯... (译注：这里作者玩了一个梗，原文 ``the future is seeing sharp，see sharp`` 发音类似 C# )

谢谢各位，大家再见。

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/06/2021-06-01_01.md)