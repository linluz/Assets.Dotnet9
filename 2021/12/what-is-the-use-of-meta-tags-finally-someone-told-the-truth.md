---
title: meta 标签到底有什么用？终于有人说出了真相
slug: what-is-the-use-of-meta-tags-finally-someone-told-the-truth
description: 最近部门在推微前端，需要按功能拆分多个子应用，主应用在加载的过程中经常出现加载失败的问题。
date: 2021-12-23 21:02:21
copyright: Reprint
author: 隐冬
originaltitle: meta 标签到底有什么用？终于有人说出了真相
originallink: https://juejin.cn/post/6987919006468407309
draft: False
cover: https://img1.dotnet9.com/2021/12/cover_34.png
categories: HTML
tags: HTML,Meta
---

## 1. 起因

最近部门在推微前端，需要按功能拆分多个子应用，主应用在加载的过程中经常出现加载失败的问题。因为https地址中，如果加载了http资源，浏览器将认为这是不安全的资源，将会默认阻止。后来在文档中添加了`<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">`完美解决。

此时我才发现自己对`meta`简直一无所知，本文主要介绍`meta`，顺带也会提一提`head`中的其它标签。如有不对请指出，最后欢迎点赞 + 收藏。

## 2. head 标签

`head`标签与`html`标签，`body`标签一样是一个文档必须的元素。

`head`标签用于定于文档头部信息，它是所有头部元素的容器。`head`中的元素可以引用脚本、指示浏览器在哪里找到样式表、提供元信息等等。

文档的头部描述了文档的各种属性和信息，包括文档的标题、在 Web 中的位置以及和其他文档的关系等。绝大多数文档头部包含的数据都不会真正作为内容显示给读者。

下面这些标签可用在 `head` 部分：`base`, `link`, `meta`, `script`, `style`, 以及 `title`。

`注意`:应该把 `head` 标签放在文档的开始处，紧跟在 `html` 后面，并处于 `body` 标签或 `frameset` 标签之前。

## 3. title 标签

`title` 定义文档的标题，它是 `head` 部分中唯一必需的元素。浏览器会以特殊的方式来使用标题，设置的内容不会显示在页面中，通常把它放置在浏览器窗口的标题栏或状态栏上，如设置为空标题展示当前页面的地址信息。

当把文档加入用户的链接列表或者收藏夹或书签列表时，标题将成为该文档链接的默认名称。

### 3.1. dir 属性

规定元素中内容的文本方向`rtl`、`ltr`。

### 3.2. lang 属性

规定元素中内容的语言代码。

## 4. meta 标签

`meta` 元素往往不会引起用户的注意，但是`meta`对整个网页有影响，会对网页能否被搜索引擎检索，和在搜索中的排名起着关键性的作用。

`meta`有个必须的属性`content`用于表示需要设置的项的值。

`meta`存在两个非必须的属性`http-equiv`和`name`, 用于表示要设置的项。

比如`<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">`,设置的项是`Content-Security-Policy`设置的值是`upgrade-insecure-requests`。

### 4.1. http-equiv 属性

`http-equiv`一般设置的都是与`http`请求头相关的信息，设置的值会关联到`http`头部。也就是说浏览器在请求服务器获取`html`的时候，服务器会将`html`中设置的`meta`放在响应头中返回给浏览器。常见的类型比如`content-type`, `expires`, `refresh`, `set-cookie`, `window-target`, `charset`， `pragma`等等。

#### 4.1.1. content-type

比如：`<meta http-equiv="content-type" content="text/html charset=utf8">`可以用来声明文档类型、设字符集，目前`content-type`只能在html文档中使用。

这样设置浏览器的头信息就会包含:

```html
content-type: text/html charset=utf8
```

#### 4.1.2. expires

用于设置浏览器的过期时间, 其实就是响应头中的expires属性。

```html
<meta http-equiv="expires" content="31 Dec 2021">
```

```shell
expires:31 Dec 2008
```

#### 4.1.3. refresh

该种设定表示5秒自动刷新并且跳转到指定的网页。如果不设置url的值那么浏览器则刷新本网页。

```html
<meta http-equiv="refresh" content="5 url=http://www.zhiqianduan.com">
```

#### 4.1.4. window-target

强制页面在当前窗口以独立页面显示, 可以防止别人在框架中调用自己的页面。

```html
<meta http-equiv="window-target" content="_top'>
```

#### 4.1.5. pragma

禁止浏览器从本地计算机的缓存中访问页面的内容

```html
<meta http-equiv="pragma" content="no-cache">
```

### 4.2. name 属性

`name`属性主要用于描述网页，与对应的`content`中的内容主要是便于搜索引擎查找信息和分类信息用的, 用法与`http-equiv`相同，`name`设置属性名，`content`设置属性值。

#### 4.2.1. author

`author`用来标注网页的作者

```html
<meta name="author" content="aaa@mail.abc.com">
```

#### 4.2.2. description

`description`用来告诉搜素引擎当前网页的主要内容，是关于网站的一段描述信息。

```html
<meta name="description" content="这是我的HTML">
```

#### 4.2.3. keywords

`keywords`设置网页的关键字，来告诉浏览器关键字是什么。是一个经常被用到的名称。它为文档定义了一组关键字。某些搜索引擎在遇到这些关键字时，会用这些关键字对文档进行分类。

```html
<meta name="keywords" content="Hello world">
```

#### 4.2.4. generator

表示当前html是用什么工具编写生成的，并没有实际作用，一般是编辑器自动创建的。

```html
<meta name="generator" content="vscode">
```

#### 4.2.5. revised

指定页面的最新版本

```html
<meta name="revised" content="V2，2015/10/1">
```

#### 4.2.6. robots

告诉搜索引擎机器人抓取哪些页面，`all / none / index / noindex / follow / nofollow`。

```html
<meta name="robots" content="all">
```

- `all`：文件将被检索，且页面上的链接可以被查询；
- `none`：文件将不被检索，且页面上的链接不可以被查询；
- `index`：文件将被检索；
- `follow`：页面上的链接可以被查询；
- `noindex`：文件将不被检索，但页面上的链接可以被查询；
- `nofollow`：文件将不被检索，页面上的链接可以被查询。

### 4.3. scheme 属性

`scheme` 属性用于指定要用来翻译属性值的方案。此方案应该在由 `head` 标签的 `profile` 属性指定的概况文件中进行了定义。`html5`不支持该属性。

## 5. base 标签

`base`标签定义了文档的基础`url`地址，在文档中所有的相对地址形式的`url`都是相对于这里定义的`url`而言的。为页面上的链接规定默认地址或目标。

```html
<base href="http://www.w3school.com.cn/i/" target="_blank" />
```

base标签包含的属性。

### 5.1. href

`href`是必选属性，指定了文档的基础`url`地址。例如，如果希望将文档的基础`URL`定义为`https：//www.abc.com`，则可以使用如下语句：`<base href="http://www.abc.com">`如果文档的超链接指向`welcom.html`,则它实际上指向的是如下`url`地址：`https://www.abc.com/welocme.html`。

### 5.2. target

定义了当文档中的链接点击后的打开方式`_blank`，`_self`，`_parrent`，`_top`。

## 6. link 标签

`link`用于引入外部样式表，在`html`的头部可以包含任意数量的`link`，`link`标签有以下常用属性。

```html
<link type="text/css" rel="stylesheet" href="github-markdown.css">
```

### 6.1. type

定义包含的文档类型，例如`text/css`

### 6.2. rel

定义`html`文档和所要包含资源之间的链接关系，可能的值有很多，最为常用的是`stylesheet`，用于包含一个固定首选样式的表单。

### 6.3. href

表示指向被包含资源的`url`地址。

## 7. style 标签

编写内部样式表的标签。

```html
<style>
    body {
        background: #f3f5f9;
    }
</style>
```

## 8. script 标签

加载`javascript`脚本的标签。加载的脚本会被默认执行。默认情况下当浏览器解析到`script`标签的时候会停止`html`的解析而开始加载`script`代码并且执行。

```html
<script src="script.js"></script>
```

### 8.1. type

指示脚本的`MIME`类型。

```html
<script type="text/javascript">
```

### 8.2. async

规定异步执行脚本，仅适用于通过`src`引入的`外部脚本`。设置的`async`属性的`script`加载不会影响后面`html`的解析，加载是与文档解析同时发生的。加载完成后立即执行。执行过程会停止`html`文档解析。

```html
<script async src="script.js"></script>
```

### 8.3. charset

规定在外部脚本文件中使用的字符编码。

```html
<script type="text/javascript" src="script.js" charset="UTF-8"></script>
```

### 8.4. defer

规定是否对脚本执行进行延迟，直到页面加载为止。设置了`defer`属性的`script`不会阻止后面`html`的解析，加载与解析是共同进行的，但是`script`的执行要在所有元素解析完成之后，`DOMContentLoaded`事件触发之前完成。

```html
<script defer src="script.js"></script>
```

### 8.5. language

规定脚本语言，与`type`功能类似，不建议使用该字段。

### 8.6. src

外部脚本的地址。

```html
<script src="script.js"></script>
```

## 9. bgsound

网站背景音乐。

```html
<bgsound src="music.mp4" autostart="true" loop="5">
```

### 9.1. src

表示背景音乐的`url`值。

### 9.2. autostart

是否自动播放`ture`自动播放，`false`不播放，默认为`false`。

### 9.3. loop

是否重复播放，值为`数字`或者`infinite`，表示重复具体次或无限次。

**参考来源**

1. [w3c](https://www.w3school.com.cn/tags/tag_head.asp)
2. [Wangkiwa](https://blog.csdn.net/qq_46580571/article/details/106035249)

>作者：隐冬
>
>链接：https://juejin.cn/post/6987919006468407309
>
>来源：稀土掘金

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/12/2021-12-23_01.md)