---
title: C# 脚本
slug: csharp-script
description: 有些情况下，需要在程序运行期间动态执行C#代码，比如，将某些经常改变的算法保存在配置文件中，在运行期间从配置文件中读取并执行运算。这时可以使用C#脚本来完成这些工作。
date: 2021-12-24 22:46:51
copyright: Reprint
author: 寻找无名的特质
originaltitle: C# 脚本
originallink: https://www.cnblogs.com/zhenl/p/15714453.html
draft: False
cover: https://img1.dotnet9.com/2021/12/cover_38.png
categories: .NET相关
tags: C#,脚本
---

有些情况下，需要在程序运行期间动态执行C#代码，比如，将某些经常改变的算法保存在配置文件中，在运行期间从配置文件中读取并执行运算。这时可以使用C#脚本来完成这些工作。

使用C#脚本需要引用库`Microsoft.CodeAnalysis.CSharp.Scripting`，下面是一些示例：

最基本的用法是计算算数表达式：

```C#
Console.Write("测试基本算数表达式:(1+2)*3/4");
var res = await CSharpScript.EvaluateAsync("(1+2)*3/4");
Console.WriteLine(res);
```

如果需要使用比较复杂的函数，可以使用`WithImports`引入名称空间：

```C#
Console.WriteLine("测试Math函数:Sqrt(2)");
res = await CSharpScript.EvaluateAsync("Sqrt(2)", ScriptOptions.Default.WithImports("System.Math"));
Console.WriteLine(res);
```

不仅是计算函数，其它函数比如`IO`，也是可以的：

```C#
Console.WriteLine(@"测试输入输出函数:Directory.GetCurrentDirectory()");
res = await CSharpScript.EvaluateAsync("Directory.GetCurrentDirectory()",
     ScriptOptions.Default.WithImports("System.IO"));
Console.WriteLine(res);
```

字符串函数可以直接调用：

```C#
Console.WriteLine(@"测试字符串函数:""Hello"".Length");
res = await CSharpScript.EvaluateAsync(@"""Hello"".Length");
Console.WriteLine(res);
```

如果需要传递变量，可以将类的实例作为上下文进行传递，下面的例子中使用了`Student`类：

```C#
Console.WriteLine(@"测试变量:");
var student = new Student { Height = 1.75M, Weight = 75 };
await CSharpScript.RunAsync("BMI=Weight/Height/Height", globals: student);
Console.WriteLine(student.BMI);
```

类`Student`:

```C#
public class Student
{
    public Decimal Height { get; set; }

    public Decimal Weight { get; set; }

    public Decimal BMI { get; set; }

    public string Status { get; set; } = string.Empty;
}
```

重复使用的脚本可以复用：

```C#
Console.WriteLine(@"测试脚本编译复用:");
var scriptBMI = CSharpScript.Create<Decimal>("Weight/Height/Height", globalsType: typeof(Student));
scriptBMI.Compile();

Console.WriteLine((await scriptBMI.RunAsync(new Student { Height = 1.72M, Weight = 65 })).ReturnValue);
```

在脚本中也可以定义函数：

```C#
Console.WriteLine(@"测试脚本中定义函数:");
string script1 = "decimal Bmi(decimal w,decimal h) { return w/h/h; } return Bmi(Weight,Height);";

var result = await CSharpScript.EvaluateAsync<decimal>(script1, globals: student);
Console.WriteLine(result);
```

在脚本中也可以定义变量：

```C#
Console.WriteLine(@"测试脚本中的变量:");
var script =  CSharpScript.Create("int x=1;");
script =  script.ContinueWith("int y=1;");
script =  script.ContinueWith("return x+y;");
Console.WriteLine((await script.RunAsync()).ReturnValue);
```

完整的实例可以从github下载：https://github.com/zhenl/CSharpScriptDemo

本文来自博客园，作者：寻找无名的特质，转载请注明原文链接：https://www.cnblogs.com/zhenl/p/15714453.html

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/12/2021-12-24_01.md)