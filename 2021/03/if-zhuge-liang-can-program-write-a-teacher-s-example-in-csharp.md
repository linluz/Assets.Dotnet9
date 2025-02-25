---
title: 如果诸葛亮用C#写出师表...
slug: if-zhuge-liang-can-program-write-a-teacher-s-example-in-csharp
description: 看到一篇18年的文章 [C++版《出师表》]，站长觉得挺有意思的，就用C# 控制台也实现了一遍
date: 2021-03-19 09:59:57
copyright: Default
originaltitle: 如果诸葛亮用C#写出师表...
draft: False
cover: https://img1.dotnet9.com/2021/03/cover_02.jpeg
categories: .NET相关
tags: C#,出师表
---

>看到一篇18年的文章 "[C++版《出师表》](https://mp.weixin.qq.com/s/7EE9kW6MiQXNB2XkLjoZDg)"，站长觉得挺有意思的，就用C# 控制台也实现了一遍，技术上没啥难度，但复制代码费了1、2个小时，纯粹无聊写着玩，看者别在意枚举、类名、变量中文命名，纯粹为了娱乐。

---

## 出师表背景（照抄原文）

今天让我们码农以特有的方式，来表达对丞相大人的敬仰与怀念！

蜀章武元年（221年），刘备称帝，诸葛亮为丞相。蜀汉建兴元年（223年），刘备病死，将刘禅托付给诸葛亮。诸葛亮实行了一系列比较正确的政治和经济措施，使蜀汉境内呈现兴旺景象。为了实现全国统一，诸葛亮在平息南方叛乱之后，于建兴五年（227年）决定北上伐魏，拟夺取魏的长安，临行之前上书后主，即这篇《出师表》。

## C# 源码

定义的一些类、枚举

```C#
public enum 王道 { 明, 昏 };

/// <summary>
/// 先帝，陛下，文景，桓灵
/// </summary>
public class 君
{
  public string 名称;
  public bool 在;
  public 王道 为君;

  public 君()
  {
    在 = true;
    为君 = 王道.明;
  }
  public bool 创业(double percentage)
  {
    if (percentage < 0.5)
    {
      在 = false;
      Console.WriteLine($"{名称}创业未半而中道崩殂！");
      return false;
    }
    else
    {
      Console.WriteLine(@"{名称}兴复汉室，还于旧都！");
      return true;
    }
  }

  public void 开张圣听()
  {
    Console.WriteLine("开张圣听，光先帝遗德！");
  }

  public void 恢弘志士之气() { }

  public void 宾自菲薄() { }

  public void 引喻失义()
  {
    Console.WriteLine("塞忠谏之路！");
  }

  public void 亲贤臣远小人()
  {
    为君 = 王道.明;
  }

  public void 亲小人远贤臣()
  {
    为君 = 王道.昏;
  }

  public void 治国() { }

  public void 偏私()
  {
    Console.WriteLine("内外异法！");
  }

  public bool 咨之(string 事)
  {
    if (王道.明 == 为君)
    {
      return true;
    }

    return false;
  }

  public bool 施行(string 事)
  {
    return true;
  }

  public void 曰(string 言)
  {
    Console.WriteLine(言);
  }

  public void 每与臣论此事()
  {
    Console.WriteLine("叹息痛恨于桓灵。");
  }

  ~君() { }
}


public enum 臣德 { 贤, 奸 }

public class 侍卫之臣
{
  private 君 刘备 = new 君();
  private 君 刘禅 = new 君();

  public string 名称;
  public 臣德 为臣;

  public void 不懈于内()
  {
    Console.WriteLine($"侍卫之臣({名称})不懈于内");
  }

  public bool 追先帝之殊遇()
  {
    if (刘备.为君 == 王道.明)
    {
      return true;
    }
    else
    {
      return false;
    }
  }

  public bool 报之于陛下()
  {
    if (刘禅.为君 == 王道.明)
    {
      return true;
    }
    else
    {
      return false;
    }
  }

  public bool 谋事(string 事)
  {
    if (为臣 == 臣德.贤)
    {
      return true;
    }

    return false;
  }
}

class 忠志之士
{
  private 君 刘备 = new 君();
  private 君 刘禅 = new 君();

  public string 名称;
  public 臣德 为臣 = 臣德.贤;

  public void 忘身于外()
  {
    Console.WriteLine($"忠志之士({名称})忘身于外!");
  }

  public bool 追先帝之殊遇()
  {
    if (刘备.为君 == 王道.明)
    {
      return true;
    }
    else
    {
      return false;
    }
  }

  public bool 报之于陛下()
  {
    if (刘禅.为君 == 王道.明)
    {
      return true;
    }
    else
    {
      return false;
    }
  }

  public bool 谋事(string 事)
  {
    if (为臣 == 臣德.贤)
    {
      return true;
    }

    return false;
  }
}

public enum 气候 { 兴盛, 疲弊, 兴隆, 倾颓 }

/// <summary>
/// 曹魏,东吴,益州,先汉,后汉
/// </summary>
public class 国
{
  public 气候 国运;

  public 国()
  {
    国运 = 气候.兴盛;
  }

  public void 付诸有司论其刑赏(侍卫之臣 臣)
  {
    if (臣.为臣 == 臣德.贤)
    {
      Console.WriteLine("赏！");
    }
    else
    {
      Console.WriteLine("刑！");
    }
  }

  ~国() { }
}

/// <summary>
/// 郭攸之，费祎
/// </summary>
public class 侍中 : 侍卫之臣 { };

/// <summary>
/// 董允
/// </summary>
public class 侍郎 : 侍卫之臣 { }

/// <summary>
/// 陈震
/// </summary>
public class 尚书 : 侍卫之臣 { }

/// <summary>
/// 张裔
/// </summary>
public class 长史 : 侍卫之臣 { }

/// <summary>
/// 蒋琬
/// </summary>
public class 参季 : 侍卫之臣 { }

/// <summary>
/// 向宠
/// </summary>
class 中都督 : 忠志之士 { }

/// <summary>
/// 诸葛亮
/// </summary>
class 丞相 : 侍卫之臣
{
  public void 回首往事()
  {
    Console.WriteLine("臣本布衣，躬耕于南阳，苟全性命于乱世，不求闻达于诸候。先帝不以臣卑鄙，猥自枉屈，三顾臣于草庐之中，咨臣以当世之事，由是感激，遂许先帝以驱驰。后值巅覆，受任于败军之际，奉命于危难之间，尔来二十有一年矣。");
  }

  public void 表忠心()
  {
    Console.WriteLine("先帝知臣谨慎，故临崩寄臣以大事也。受命以来，夙夜忧叹，恐托付不效，以伤先帝之明。故五月渡泸，深入不毛。");
  }

  public void 请战()
  {
    Console.WriteLine("今南方已定，兵甲已足，当奖率三军，北定中原，庶竭驽钝，攘除奸凶，兴复汉室, 还于旧都。");
  }
  public void 道别()
  {
    Console.WriteLine("今当远离, 临表涕零, 不知所言。");
  }
}
```

Main方法

```C#
static void Main(string[] args)
{
  君 先帝 = new 君();
  先帝.名称 = "先帝";
  先帝.创业(0.49);

  国 益州 = new 国();
  益州.国运 = 气候.疲弊;

  Console.WriteLine("此诚危急存亡之秋也！");

  侍中 郭攸之 = new 侍中();
  郭攸之.名称 = "郭攸之";
  if (郭攸之.追先帝之殊遇() && 郭攸之.报之于陛下())
  {
    郭攸之.不懈于内();
  }

  侍中 费祎 = new 侍中();
  费祎.名称 = "费祎";
  if (费祎.追先帝之殊遇() && 费祎.报之于陛下())
  {
    费祎.不懈于内();
  }

  侍郎 董允 = new 侍郎();
  董允.名称 = "董允";
  if (董允.追先帝之殊遇() && 董允.报之于陛下())
  {
    董允.不懈于内();
  }

  中都督 向宠 = new 中都督();
  向宠.名称 = "向宠";
  if (向宠.追先帝之殊遇() && 向宠.报之于陛下())
  {
    向宠.忘身于外();
  }

  君 陛下 = new 君();
  if (陛下.为君 == 王道.明)
  {
    陛下.开张圣听();
    陛下.恢弘志士之气();
  }
  else
  {
    陛下.宾自菲薄();
    陛下.引喻失义();
  }

  陛下.治国();

  bool 宫中 = false;
  bool 府中 = false;
  bool 陟臧 = false;
  bool 罚否 = false;
  Debug.Assert(宫中 == 府中);
  Debug.Assert(陟臧 == 罚否);

  侍卫之臣 作奸犯科者 = new 侍卫之臣();
  作奸犯科者.为臣 = 臣德.奸;
  侍卫之臣 为忠善者 = new 侍卫之臣();
  为忠善者.为臣 = 臣德.贤;
  if (陛下.为君 == 王道.明)
  {
    益州.付诸有司论其刑赏(作奸犯科者);
    益州.付诸有司论其刑赏(为忠善者);
  }
  else
  {
    陛下.偏私();
  }

  if (郭攸之.为臣 == 臣德.贤
    && 费祎.为臣 == 臣德.贤
    && 董允.为臣 == 臣德.贤)
  {
    Console.WriteLine("此皆良实，志虑忠纯，是以先帝简拔以遗陛下。");
  }

  string 宫中之事 = null;
  if (陛下.咨之(宫中之事)
    && 郭攸之.谋事(宫中之事)
    && 费祎.谋事(宫中之事)
    && 董允.谋事(宫中之事))
  {

    陛下.施行(宫中之事);
    Console.WriteLine("裨补阙病, 有所广益");
  }

  if (向宠.为臣 == 臣德.贤)
  {
    Console.WriteLine("性行淑均，晓畅军事。");
    Console.Write("先帝称之曰：");
    先帝.曰("能");
    Console.WriteLine("是以众议举宠为督。");
  }

  string 营中之事 = null;
  if (陛下.咨之(营中之事))
  {
    陛下.施行(宫中之事);
    Console.WriteLine("行阵和睦，优劣得所！");
  }

  君 文景 = new 君();
  君 恒灵 = new 君();
  国 先汉 = new 国();
  国 后汉 = new 国();

  文景.亲贤臣远小人();
  先汉.国运 = 气候.兴隆;
  恒灵.亲小人远贤臣();
  后汉.国运 = 气候.倾颓;

  do
  {
    先帝.每与臣论此事();
  } while (先帝.在);

  if (郭攸之.为臣 == 臣德.贤
          && 费祎.为臣 == 臣德.贤
          && 董允.为臣 == 臣德.贤)
  {
    Console.WriteLine("此悉贞良死节之臣，愿陛下亲之信之，则汉室之隆，可计日而待也。");
  }

  丞相 诸葛亮 = new 丞相();
  诸葛亮.回首往事();
  诸葛亮.表忠心();
  诸葛亮.请战(); // 此臣所以报先帝而忠陛下之职分也
  诸葛亮.道别();

}
```

## 代码输出《出师表》

![出师表部分输出](https://img1.dotnet9.com/2021/03/0201.jpeg)


```shell
先帝创业未半而中道崩殂！
此诚危急存亡之秋也！
侍卫之臣(郭攸之)不懈于内
侍卫之臣(费祎)不懈于内
侍卫之臣(董允)不懈于内
忠志之士(向宠)忘身于外!
开张圣听，光先帝遗德！
刑！
赏！
此皆良实，志虑忠纯，是以先帝简拔以遗陛下。
裨补阙病, 有所广益
性行淑均，晓畅军事。
先帝称之曰：能
是以众议举宠为督。
行阵和睦，优劣得所！
叹息痛恨于桓灵。
此悉贞良死节之臣，愿陛下亲之信之，则汉室之隆，可计日而待也。
臣本布衣，躬耕于南阳，苟全性命于乱世，不求闻达于诸候。先帝不以臣卑鄙，猥自枉屈，三顾臣于草庐之中，咨臣以当世之事，由是感激，遂许先帝以驱驰。后值巅覆，受任于败军之际，奉命于危难之间，尔来二十有一年矣。
先帝知臣谨慎，故临崩寄臣以大事也。受命以来，夙夜忧叹，恐托付不效，以伤先帝之明。故五月渡泸，深入不毛。
今南方已定，兵甲已足，当奖率三军，北定中原，庶竭驽钝，攘除奸凶，兴复汉室, 还于旧都。
今当远离, 临表涕零, 不知所言。
```

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/03/2021-03-19_02.md)