---
title: ASP.NET Core WebApi返回结果统一包装实践
slug: ASP-dotNET-Core-WebApi-return-result-unified-packaging-practice
description: 关于WebApi统一结果返回的时候，让我也有了更一步的思考，首先是如何能更好的限制返回统一的格式，其次是关于结果的包装一定是更简单更强大。在不断的思考和完善中，终于有了初步的成果，便分享出来，学无止境思考便无止境，希望以此能与君共勉。
date: 2022-04-13 07:12:36
copyright: Reprint
author: yi念之间
originaltitle: ASP.NET Core WebApi返回结果统一包装实践
originallink: https://www.cnblogs.com/wucy/p/16124449.html
draft: False
cover: https://img1.dotnet9.com/2022/04/cover_12.jpg
categories: Web API
---

## 前言

近期在重新搭建一套基于ASP.NET Core WebAPI的框架，这其中确实带来了不少的收获，毕竟当你想搭建一套框架的时候，你总会不自觉的去想，如何让这套框架变得更完善一点更好用一点。其中在关于WebApi统一结果返回的时候，让我也有了更一步的思考，首先是如何能更好的限制返回统一的格式，其次是关于结果的包装一定是更简单更强大。在不断的思考和完善中，终于有了初步的成果，便分享出来，学无止境思考便无止境，希望以此能与君共勉。

## 统一结果类封装

首先如果让返回的结果格式统一，就得有一个统一的包装类去包装所有的返回结果，因为返回的具体数据虽然格式一致，但是具体的值的类型是不确定的，因此我们这里需要定义个泛型类。当然如果你不选择泛型类的话用dynamic或者object类型也是可以的,但是这样的话可能会带来两点不足

- 一是可能会存在装箱拆箱的操作。
- 二是如果引入swagger的话是没办法生成返回的类型的，因为dynamic或object类型都是执行具体的action时才能确定返回类型的，但是swagger的结构是首次运行的时候就获取到的，因此无法感知具体类型。

## 定义包装类

上面我们也说了关于定义泛型类的优势，这里就话不多说来直接封装一个结果返回的包装类

```C#
public class ResponseResult<T>
{
    /// <summary>
    /// 状态结果
    /// </summary>
    public ResultStatus Status { get; set; } = ResultStatus.Success;

    private string? _msg;

    /// <summary>
    /// 消息描述
    /// </summary>
    public string? Message
    {
        get
        {
            // 如果没有自定义的结果描述，则可以获取当前状态的描述
            return !string.IsNullOrEmpty(_msg) ? _msg : EnumHelper.GetDescription(Status);
        }
        set
        {
            _msg = value;
        }
    }

    /// <summary>
    /// 返回结果
    /// </summary>
    public T Data { get; set; }
}
```

其中这里的`ResultStatus`是一个枚举类型，用于定义具体的返回状态码，用于判断返回的结果是正常还是异常或者其他，我这里只是简单的定义了一个最简单的示例，有需要的话也可以自行扩展

```C#
public enum ResultStatus
{
    [Description("请求成功")]
    Success = 1,
    [Description("请求失败")]
    Fail = 0,
    [Description("请求异常")]
    Error = -1
}
```

这种情况下定义枚举类型并且结合它的`DescriptionAttribute`的特性去描述枚举的含义是一个不错的选择，首先它可以统一管理每个状态的含义，其次是更方便的获取每个状态对应的描述。这样的话如果没有自定义的结果描述，则可以获取当前状态的描述来充当默认值的情况。这个时候在写具体action的时候会是以下的效果

```C#
[HttpGet("GetWeatherForecast")]
public ResponseResult<IEnumerable<WeatherForecast>> GetAll()
{
    var datas = Enumerable.Range(1, 5).Select(index => new WeatherForecast
    {
        Date = DateTime.Now.AddDays(index),
        TemperatureC = Random.Shared.Next(-20, 55),
        Summary = Summaries[Random.Shared.Next(Summaries.Length)]
    });
    return new ResponseResult<IEnumerable<WeatherForecast>> {  Data = datas };
}
```

这样的话每次编写action的时候都可以返回一个`ResponseResult<T>`的结果了，这里就体现出了使用枚举定义状态码的优势了，相当一部分场景我们可以省略了状态码甚至是消息的编写，毕竟很多时候在保障功能的情况下，代码还是越简介越好的，更何况是一些高频操作呢。

## 升级一下操作

上面虽然我们定义了`ResponseResult<T>`来统一包装返回结果，但是每次还得new一下，在无疑是不太方便的，而且还要每次都还得给属性赋值啥的，也是够麻烦的，这个时候就想，如果能有相关的辅助方法去简化操作就好了，方法不用太多能满足场景就好，也就是够用就好，最主要的是能支持扩展就可以。因此，进一步升级一下结果包装类，来简化一下操作

```C#
public class ResponseResult<T>
{
    /// <summary>
    /// 状态结果
    /// </summary>
    public ResultStatus Status { get; set; } = ResultStatus.Success;

    private string? _msg;

    /// <summary>
    /// 消息描述
    /// </summary>
    public string? Message
    {
        get
        {
            return !string.IsNullOrEmpty(_msg) ? _msg : EnumHelper.GetDescription(Status);
        }
        set
        {
            _msg = value;
        }
    }

    /// <summary>
    /// 返回结果
    /// </summary>
    public T Data { get; set; }

    /// <summary>
    /// 成功状态返回结果
    /// </summary>
    /// <param name="result">返回的数据</param>
    /// <returns></returns>
    public static ResponseResult<T> SuccessResult(T data)
    {
        return new ResponseResult<T> { Status = ResultStatus.Success, Data = data };
    }

    /// <summary>
    /// 失败状态返回结果
    /// </summary>
    /// <param name="code">状态码</param>
    /// <param name="msg">失败信息</param>
    /// <returns></returns>
    public static ResponseResult<T> FailResult(string? msg = null)
    {
        return new ResponseResult<T> { Status = ResultStatus.Fail, Message = msg };
    }

    /// <summary>
    /// 异常状态返回结果
    /// </summary>
    /// <param name="code">状态码</param>
    /// <param name="msg">异常信息</param>
    /// <returns></returns>
    public static ResponseResult<T> ErrorResult(string? msg = null)
    {
        return new ResponseResult<T> { Status = ResultStatus.Error, Message = msg };
    }

    /// <summary>
    /// 自定义状态返回结果
    /// </summary>
    /// <param name="status"></param>
    /// <param name="result"></param>
    /// <returns></returns>
    public static ResponseResult<T> Result(ResultStatus status, T data, string? msg = null)
    {
        return new ResponseResult<T> { Status = status, Data = data, Message = msg };
    }
}
```

这里进一步封装了几个方法，至于具体封装几个这种方法，还是那句话够用就好，这里我封装了几个常用的操作，成功状态、失败状态、异常状态、最完全状态，这几种状态基本上可以满足大多数的场景，不够的话可以自行进行进一步的多封装几个方法。这样的话在action使用的时候就会简化很多,省去了手动属性赋值

```C#
[HttpGet("GetWeatherForecast")]
public ResponseResult<IEnumerable<WeatherForecast>> GetAll()
{
    var datas = Enumerable.Range(1, 5).Select(index => new WeatherForecast
    {
        Date = DateTime.Now.AddDays(index),
        TemperatureC = Random.Shared.Next(-20, 55),
        Summary = Summaries[Random.Shared.Next(Summaries.Length)]
    });
    return ResponseResult<IEnumerable<WeatherForecast>>.SuccessResult(datas);
}
```

## 进一步完善

上面我们通过完善`ResponseResult<T>`类的封装，确实在某些程度上节省了一部分操作，但是还是有点美中不足，那就是每次返回结果的时候，虽然定义了几个常用的静态方法去操作返回结果，但是每次还得通过手动去把`ResponseResult<T>`类给请出来才能使用，现在呢想在操作返回结果的时候不想看到它了。这个呢也很简单，我们可以借助微软针对MVC的Controller的封装进一步得到一个思路，那就是定义一个基类的Controller，我们在Controller基类中，把常用的返回结果封装一些方法，这样在Controller的子类中呢就可以直接调用这些方法，而不需要关注如何去编写方法的返回类型了，话不多说动手把Controller基类封装起来

```C#
[ApiController]
[Route("api/[controller]")]
public class ApiControllerBase : ControllerBase
{
    /// <summary>
    /// 成功状态返回结果
    /// </summary>
    /// <param name="result">返回的数据</param>
    /// <returns></returns>
    protected ResponseResult<T> SuccessResult<T>(T result)
    {
        return ResponseResult<T>.SuccessResult(result);
    }

    /// <summary>
    /// 失败状态返回结果
    /// </summary>
    /// <param name="code">状态码</param>
    /// <param name="msg">失败信息</param>
    /// <returns></returns>
    protected ResponseResult<T> FailResult<T>(string? msg = null)
    {
        return ResponseResult<T>.FailResult(msg);
    }

    /// <summary>
    /// 异常状态返回结果
    /// </summary>
    /// <param name="code">状态码</param>
    /// <param name="msg">异常信息</param>
    /// <returns></returns>
    protected ResponseResult<T> ErrorResult<T>(string? msg = null)
    {
        return ResponseResult<T>.ErrorResult(msg);
    }

    /// <summary>
    /// 自定义状态返回结果
    /// </summary>
    /// <param name="status"></param>
    /// <param name="result"></param>
    /// <returns></returns>
    protected ResponseResult<T> Result<T>(ResultStatus status, T result, string? msg = null)
    {
        return ResponseResult<T>.Result(status, result, msg);
    }
}
```

有了这么一个大神的辅助，一切似乎又向着美好进发了一步，这样的话每次我们自定义的Controller可以继承ApiControllerBase类，从而使用里面的简化操作。所以再写起来代码，大概是这么一个效果

```C#
public class WeatherForecastController : ApiControllerBase
{
    private static readonly string[] Summaries = new[]
    {
       "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };

    [HttpGet("GetWeatherForecast")]
    public ResponseResult<IEnumerable<WeatherForecast>> GetAll()
    {
        var datas = Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateTime.Now.AddDays(index),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)]
        });
        return SuccessResult(datas);
    }
}
```

这个时候确实变得很美好了，但是还是没有逃脱一点，那就是我还是得通过特定的方法来得到一个`ResponseResult<T>`类型的返回结果，包括我们给`ResponseResult<T>`类封装静态方法，或者甚至是定义`ApiControllerBase`基类，都是为了进一步简化这个操作。现在呢我想告别这个限制，我能不能把返回的结果直接就默认的转化成`ResponseResult<T>`类型的结果呢？当然可以，这也是通过ASP.NET Core的封装思路中得到的启发，借助`implicit`自动完成隐式转换，这个在ASP.NET Core的`ActionResult<T>`类中也有体现

```C#
public static implicit operator ActionResult<TValue>(TValue value)
{
    return new ActionResult<TValue>(value);
}
```

通过这个思路我们可以进一步完善`ResponseResult<T>`类的实现方式，给它添加一个隐式转换的操作，仅仅定义一个方法即可，在`ResponseResult<T>`类中继续完善

```C#
/// <summary>
/// 隐式将T转化为ResponseResult<T>
/// </summary>
/// <param name="value"></param>
public static implicit operator ResponseResult<T>(T value)
{
    return new ResponseResult<T> { Data = value };
}
```

这种对于绝大部分返回成功结果的时候提供了非常简化的操作，这个时候如果你再去使用action的时候就可以进一步来简化返回值的操作了

```C#
[HttpGet("GetWeatherForecast")]
public ResponseResult<IEnumerable<WeatherForecast>> GetAll()
{
    var datas = Enumerable.Range(1, 5).Select(index => new WeatherForecast
    {
        Date = DateTime.Now.AddDays(index),
        TemperatureC = Random.Shared.Next(-20, 55),
        Summary = Summaries[Random.Shared.Next(Summaries.Length)]
    });
    return datas.ToList();
}
```

因为我们定义了`T`到`ResponseResult<T>`的隐式转换，所以这个时候我们就可以直接返回结果了，而不需要手动对结果返回值进行包装。

## 漏网之鱼处理

在上面我们为了尽量简化action返回`ResponseResult<T>`的统一返回结构的封装，已经对`ResponseResult<T>`类进行了许多的封装，并且还通过封装`ApiControllerBase`基类进一步简化这一操作，但是终究还是避免不了一点，那就是很多时候可能想不起来对action的返回值去加`ResponseResult<T>`类型的返回值，但是我们之前的所有封装都得建立在必须要声明`ResponseResult<T>`类型的返回值的基础上才行，否则就不存在统一返回格式这一说法了。所以针对这些漏网之鱼，我们必须要有统一的拦截机制，这样才能更完整的针对返回结果进行处理，针对这种对action返回值的操作，我们首先想到的就是定义`过滤器`进行处理，因此笔者针对这一现象封装了一个统一包装结果的过滤器，实现如下

```C#
public class ResultWrapperFilter : ActionFilterAttribute
{
    public override void OnResultExecuting(ResultExecutingContext context)
    {
        var controllerActionDescriptor = context.ActionDescriptor as ControllerActionDescriptor;
        var actionWrapper = controllerActionDescriptor?.MethodInfo.GetCustomAttributes(typeof(NoWrapperAttribute), false).FirstOrDefault();
        var controllerWrapper = controllerActionDescriptor?.ControllerTypeInfo.GetCustomAttributes(typeof(NoWrapperAttribute), false).FirstOrDefault();
        //如果包含NoWrapperAttribute则说明不需要对返回结果进行包装，直接返回原始值
        if (actionWrapper != null || controllerWrapper != null)
        {
            return;
        }

        //根据实际需求进行具体实现
        var rspResult = new ResponseResult<object>();
        if (context.Result is ObjectResult)
        {
            var objectResult = context.Result as ObjectResult;
            if (objectResult?.Value == null)
            {
                rspResult.Status = ResultStatus.Fail;
                rspResult.Message = "未找到资源";
                context.Result = new ObjectResult(rspResult);
            }
            else
            {
                //如果返回结果已经是ResponseResult<T>类型的则不需要进行再次包装了
                if (objectResult.DeclaredType.IsGenericType && objectResult.DeclaredType?.GetGenericTypeDefinition() == typeof(ResponseResult<>))
                {
                    return;
                }
                rspResult.Data = objectResult.Value;
                context.Result = new ObjectResult(rspResult);
            }
            return;
        }
    }
}
```

在使用WebAPI的过程中，我们的action绝大部分是直接返回`ViewModel`或`Dto`而并没有返回`ActionResult`类型相关，但是无妨，这个时候MVC的底层操作会为我们将这些自定义的类型包装成`ObjectResult`类型的，因此我们的`ResultWrapperFilter`过滤器也是通过这一机制进行操作的。这里有两点需要考虑的

- 首先是，我们必须要允许并非所有的返回结果都要进行`ResponseResult<T>`的包装，为了满足这一需求我们还定义了`NoWrapperAttribute`来实现这一效果，只要Controller或Action有`NoWrapperAttribute`的修饰则不对返回结果进行任何处理。
- 其次是，如果我们的Action上的返回类型已经是`ResponseResult<T>`类型的，则也不需要对返回结果进行再次的包装。

关于`ResultWrapperFilter`的定义其实很简单，因为在这里它只是起到了一个标记的作用

```C#
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = false, Inherited = false)]
public class NoWrapperAttribute:Attribute
{
}
```

到了这里，还有一种特殊的情况需要注意，那就是当程序发生异常的时候，我们上面的这些机制也是没有办法生效的，因此我们还需要定义一个针对全局异常处理的拦截机制，同样是可以使用统一异常处理过滤器进行操作，实现如下

```C#
public class GlobalExceptionFilter : IExceptionFilter
{
    private readonly ILogger<GlobalExceptionFilter> _logger;
    public GlobalExceptionFilter(ILogger<GlobalExceptionFilter> logger)
    {
        _logger = logger;
    }

    public void OnException(ExceptionContext context)
    {
        //异常返回结果包装
        var rspResult = ResponseResult<object>.ErrorResult(context.Exception.Message);
        //日志记录
        _logger.LogError(context.Exception, context.Exception.Message);
        context.ExceptionHandled = true;
        context.Result = new InternalServerErrorObjectResult(rspResult);
    }

    public class InternalServerErrorObjectResult : ObjectResult
    {
        public InternalServerErrorObjectResult(object value) : base(value)
        {
            StatusCode = StatusCodes.Status500InternalServerError;
        }
    }
}
```

写完过滤器了，千万不能忘了全局注册一下，否则它也就只能看看了，不会起到任何效果

```C#
builder.Services.AddControllers(options =>
{
    options.Filters.Add<ResultWrapperFilter>();
    options.Filters.Add<GlobalExceptionFilter>();
});
```

## 漏网之鱼另一种处理

当然针对上面两种针对漏网之鱼的处理，在ASP.NET Core上还可以通过中间件的方式进行处理，至于过滤器和中间件有何不同，相信大家已经非常清楚了，核心不同总结起来就一句话

>二者的处理阶段不同，即针对管道的生命周期处理是不一样的，中间件可以处理任何生命周期在它之后的场景，但是过滤器只管理Controller这一块的一亩三分地

但是针对结果包装这一场景，笔者觉得使用过滤器的方式更容易处理一点，因为毕竟我们是要操作Action的返回结果，通过过滤器中我们可以直接拿到返回结果的值。但是这个操作如果在中间件里进行操作的话，只能通过读取`Response.Body`进行操作了，笔者这里也封装了一个操作，如下所示

```C#
public static IApplicationBuilder UseResultWrapper(this IApplicationBuilder app)
{
        var serializerOptions = app.ApplicationServices.GetRequiredService<IOptions<JsonOptions>>().Value.JsonSerializerOptions;
        serializerOptions.Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping;
        return app.Use(async (context, next) =>
        {
            var originalResponseBody = context.Response.Body;
            try
            {
                //因为Response.Body没办法进行直接读取，所以需要特殊操作一下
                using var swapStream = new MemoryStream();
                context.Response.Body = swapStream;
                await next();
                //判断是否出现了异常状态码，需要特殊处理
                if (context.Response.StatusCode == StatusCodes.Status500InternalServerError)
                {
                    context.Response.Body.Seek(0, SeekOrigin.Begin);
                    await swapStream.CopyToAsync(originalResponseBody);
                    return;
                }
                var endpoint = context.Features.Get<IEndpointFeature>()?.Endpoint;
                if (endpoint != null)
                {
                    //只针对application/json结果进行处理
                    if (context.Response.ContentType.ToLower().Contains("application/json"))
                    {
                        //判断终结点是否包含NoWrapperAttribute
                        NoWrapperAttribute noWrapper = endpoint.Metadata.GetMetadata<NoWrapperAttribute>();
                        if (noWrapper != null)
                        {
                            context.Response.Body.Seek(0, SeekOrigin.Begin);
                            await swapStream.CopyToAsync(originalResponseBody);
                            return;
                        }
                        //获取Action的返回类型
                        var controllerActionDescriptor = context.GetEndpoint()?.Metadata.GetMetadata<ControllerActionDescriptor>();
                        if (controllerActionDescriptor != null)
                        {
                            //泛型的特殊处理
                            var returnType = controllerActionDescriptor.MethodInfo.ReturnType;
                            if (returnType.IsGenericType && (returnType.GetGenericTypeDefinition() == typeof(Task<>) || returnType.GetGenericTypeDefinition() == typeof(ValueTask<>)))
                            {
                                returnType = returnType.GetGenericArguments()[0];
                            }
                            //如果终结点已经是ResponseResult<T>则不进行包装处理
                            if (returnType.IsGenericType && returnType.GetGenericTypeDefinition() == typeof(ResponseResult<>))
                            {
                                context.Response.Body.Seek(0, SeekOrigin.Begin);
                                await swapStream.CopyToAsync(originalResponseBody);
                                return;
                            }
                            context.Response.Body.Seek(0, SeekOrigin.Begin);
                            //反序列化得到原始结果
                            var result = await JsonSerializer.DeserializeAsync(context.Response.Body, returnType, serializerOptions);
                            //对原始结果进行包装
                            var bytes = JsonSerializer.SerializeToUtf8Bytes(ResponseResult<object>.SuccessResult(result), serializerOptions);
                            new MemoryStream(bytes).CopyTo(originalResponseBody);
                            return;
                        }
                    }
                }
                context.Response.Body.Seek(0, SeekOrigin.Begin);
                await swapStream.CopyToAsync(originalResponseBody);
            }
            finally
            {
                //将原始的Body归还回来
                context.Response.Body = originalResponseBody;
            }
        });
    }
}
```

相信通过上面的处理，我们就可以更容易的看出来，谁更容易的对统一结果进行包装处理了，毕竟我们是针对Action的返回结果进行处理，而过滤器显然就是为针对Controller和Action的处理而生的。但是通过中间件的方式能更完整的针对结果进行处理，因为许多时候我们可能是在自定义的中间件里直接拦截请求并返回，但是根据二八原则这种情况相对于Action的返回值毕竟是少数，有这种情况我们可以通过直接`ResponseResult<T>`封装的方法进行返回操作，也很方便。但是这个时候呢，关于异常处理我们通过全局异常处理中间件，则能更多的处理更多的场景，且没有副作用，看一下它的定义

```C#
public static IApplicationBuilder UseException(this IApplicationBuilder app)
{
    return app.UseExceptionHandler(configure =>
    {
        configure.Run(async context =>
        {
            var exceptionHandlerPathFeature = context.Features.Get<IExceptionHandlerPathFeature>();
            var ex = exceptionHandlerPathFeature?.Error;
            if (ex != null)
            {
                var _logger = context.RequestServices.GetService<ILogger<IExceptionHandlerPathFeature>>();
                var rspResult = ResponseResult<object>.ErrorResult(ex.Message);
                _logger?.LogError(ex, message: ex.Message);
                context.Response.StatusCode = StatusCodes.Status500InternalServerError;
                context.Response.ContentType = "application/json;charset=utf-8";
                await context.Response.WriteAsync(rspResult.SerializeObject());
            }
        });
    });
}
```

使用全局异常梳理中间件是没有副作用的，主要因为在异常处理的时候我们不需要读取Response.Body进行读取操作的，所以至于是选择异常处理中间件还是过滤器，大家可以针对自己的实际场景进行选择，两种方式都是可以的。

## 总结

本文主要是展示了针对ASP.NET Core WeApi结果统一返回格式的相关操作，通过示例我们一步一步的展示了完成这一目标的不断升级的实现，虽然整体看起来比较简单，但是却承载着笔者一次又一次的思考升级。每次实现完一个阶段，都会去想有没有更好的方式去完善它。这其中还有一些思路来自微软源码为我们提供的思路，所以很多时候还是建议大家去看一看源码的，可以在很多时候为我们提供一种解决问题的思路。正如我看到的一句话，读源码也是一种围城，外面的人不想进去，里面的人不想出来。如果大家有更好的实现方式，欢迎一起讨论。曾经的时候我会为自己学到了一个新的技能而感到高兴，到了后来我会对有一个好的思路，或者好的解决问题的方法而感到高兴。读万卷书很重要，行万里路同样重要，读书是沉淀，行路是实践，结合到一起才能更好的促进，而不是只选择一种。


👇欢迎扫码关注我的公众号👇

![](https://img1.dotnet9.com/2022/04/1201.png)