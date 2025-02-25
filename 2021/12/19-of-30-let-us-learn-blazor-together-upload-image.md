---
title: (19/30)大家一起学Blazor：图片上传
slug: 19-of-30-let-us-learn-blazor-together-upload-image
description: 在大部分的网站中，上传图片也是很重要的功能，今天我们就来操作下。
date: 2021-12-21 22:04:11
copyright: Reprint
author: StrayaWorker
originaltitle: (19/30)大家一起学Blazor：图片上传
originallink: https://ithelp.ithome.com.tw/articles/10267909
draft: False
cover: https://img1.dotnet9.com/2021/12/cover_05.png
albums: 学Blazor
categories: Blazor
tags: Blazor Server
---

在大部分的网站中，上传图片也是很重要的功能，今天我们就来操作下。

**(注：这是用Blazor Server 的方式，但最好不要上传太多文件，所以限制上传4张照片的话就会提示，毕竟这些事都是在服务器上做，负担太大，微软也建议用.NET Core Web API 的方式操作)**

我们先建立一个Component `FileUpload`。

下面代码为`FileUpload.razor`，使用Blazor 提供的Component `<InputFile>`，`multiple`代表可以上传多个文件

```C#
@page "/FileUpload"

<div>
    <div>
        <InputFile OnChange="OnChange" multiple></InputFile>
    </div>
    <div>
        <MyButton value="Submit" class="btn btn-primary" type="submit" @onclick="OnSubmit"></MyButton>
    </div>
</div>
@if (ImageList.Count > 0)
{
    <table>
        <tr>
            @foreach (var img in ImageList)
            {
                <td>
                    <img src="@img" width="150" height="150"/>
                </td>
            }
        </tr>
    </table>
}
```

下面代码为`FileUpload.razor.cs`，这里用`partial` class

```C#
using BlazorServer.ViewModels;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Forms;
using Microsoft.JSInterop;
using System.Text.Json;

namespace BlazorServer.Shared;

public partial class FileUpload
{
	private JsInteropClasses? _jsClass;

	// 取得`<InputFile>`的文件内容
	public IReadOnlyList<IBrowserFile>? ImageFiles;

	public List<string> ImageList = new();
	public string? ImageSrc;
	[Inject] protected IJSRuntime Js { get; set; }

	/// <summary>
	///     用以判断runtime期间在什么环境执行
	/// </summary>
	[Inject]
	protected IWebHostEnvironment? Env { get; set; }

	protected override Task OnInitializedAsync()
	{
		_jsClass = new JsInteropClasses(Js);
		return base.OnInitializedAsync();
	}

	public async Task OnChange(InputFileChangeEventArgs e)
	{
		ImageList = new List<string>();
		const string format = "image/jpeg";

		// 取得文件
		ImageFiles = e.GetMultipleFiles();
		foreach (var file in ImageFiles)
		{
			// 将图片内容转换成指定类型及最大尺寸
			var imageFile = await file.RequestImageFileAsync(format, 1200, 675);

			// 利用Stream读取图片内容
			await using var fileStream = imageFile.OpenReadStream();

			// 将 Stream读取到内存中，如果没有确定要上传不要这么做，以免浪费内存
			await using var memoryStream = new MemoryStream();
			await fileStream.CopyToAsync(memoryStream);
			ImageSrc = $"data:{format};base64,{Convert.ToBase64String(memoryStream.ToArray())}";

			// 以Data URI的方式将图片呈现
			ImageList.Add(ImageSrc);
		}
	}

	public async Task OnSubmit()
	{
		// 将提示信息变成ViewModel
		var sweetConfirm = new SweetConfirmViewModel
		{
			RequestTitle = "是否确定上传图片？",
			ResponseTitle = "上传成功"
		};
		var jsonString = JsonSerializer.Serialize(sweetConfirm);
		var result = await _jsClass!.Confirm(jsonString);
		if (result && ImageFiles != null && ImageFiles.Any())
		{
			long maxFileSize = 1024 * 1024 * 15;

			// 指定图片存储路径
			var folder = $@"{Env!.WebRootPath}\images";
			foreach (var file in ImageFiles)
			{
				// 使用Stream 将文件存储到指定路径
				await using var stream = file.OpenReadStream(maxFileSize);

				//如果文件夹不存在先创建
				Directory.CreateDirectory(folder);

				var path = $@"{Env.WebRootPath}\images\{file.Name}";

				//创建文件
				var fs = File.Create(path);

				// 将图片Stream复制到文件中
				await stream.CopyToAsync(fs);

				// Stream用完记得关闭
				stream.Close();
				fs.Close();
			}
		}
	}
}
```

为了方便，`NavMenu.razor`加上路由可跳转这个Component

```html
<div class="nav-item px-3">
    <NavLink class="nav-link" href="FileUpload" Match="NavLinkMatch.All">
        <span class="bi bi-card-image h4 p-2 mb-0" aria-hidden="true"></span> File Upload
    </NavLink>
</div>
```

建立新的`ViewModel` 让`SweetConfirm`可以通用

```C#
namespace BlazorServer.ViewModels;

public class SweetConfirmViewModel
{
	public string? ResponseTitle { get; set; }

	public string? ResponseText { get; set; }

	public string? RequestTitle { get; set; }

	public string? RequestText { get; set; }
}
```

再把`_Layout.cshtml`的`SweetConfirm`修改一下

```JavaScript
function SweetConfirm(jsonString) {
    // 这里要parse才能正常传回来
    var arg = JSON.parse(jsonString);
    return new Promise((resolve) => {
        swal({
            title: arg.RequestTitle,
            text: arg.RequestText,
            icon: "warning",
            buttons: ["取消", "确定"],
            dangerMode: true
        }).then((willDelete) => {
            resolve(willDelete);
            if (willDelete) {
                swal(arg.ResponseTitle, arg.ResponseText, "success");
            }
        });

    });
}
```
        
既然这边改了，`PostBase.razor.cs`的`DeletePost`也要修改

```C#
protected async Task DeletePost()
{
    // 改成ViewModel
    var sweetConfirm = new SweetConfirmViewModel
    {
        RequestTitle = $"是否确定删除日志 {Post!.Title}?",
        RequestText = "这个操作不可恢复",
        ResponseTitle = "删除成功",
        ResponseText = "日志被删除了"
    };
    var jsonString = JsonSerializer.Serialize(sweetConfirm);
    var result = await _jsClass.Confirm(jsonString);
    if (result)
    {
        var deleted = await PostRepository!.DeletePost(Post.Id);
        if (deleted.IsSuccess)
            await GetPostId.InvokeAsync(Post!.Id);
        else
            await _jsClass.Alert(deleted.Message!);
    }
}
```
        
`JsInteropClasses.cs`的`Confirm()`改成JSON 字串

```C#
public async ValueTask<bool> Confirm(string jsonString)
{
    var confirm = await _js.InvokeAsync<object?>("SweetConfirm", jsonString);
    if (confirm == null)
        return false;
    return bool.TryParse(confirm.ToString(), out var result) && result;
}
```

可以看到图片上传成功了

![](https://img1.dotnet9.com/2021/12/2901.gif)

**引用：**

1. [ASP.NET Core Blazor file uploads](https://docs.microsoft.com/en-us/aspnet/core/blazor/file-uploads?view=aspnetcore-5.0&pivots=server)
2. [Upload Files Using InputFile Component In Blazor](http://www.binaryintellect.net/articles/06473cc7-a391-409e-948d-3752ba3b4a6c.aspx)
3. [What scope does a using statement have without curly braces](https://stackoverflow.com/a/24819614)
4. [BrowserFileExtensions.RequestImageFileAsync(IBrowserFile, String, Int32, Int32) 方法](https://docs.microsoft.com/zh-tw/dotnet/api/microsoft.aspnetcore.components.forms.browserfileextensions.requestimagefileasync?view=aspnetcore-5.0)
5. [Day 26：Blazor WebAssembly 上传文件](https://ithelp.ithome.com.tw/articles/10251852)

**注：本文代码通过 .NET 6 + Visual Studio 2022重构，可点击原文链接与重构后代码比较学习，谢谢阅读，支持原作者**

- 本文Markdown：[点击浏览](https://github.com/dotnet9/Assets.Dotnet9/blob/main/2021/12/2021-12-21_02.md)