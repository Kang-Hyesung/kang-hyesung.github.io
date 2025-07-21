---
title: "[4] 커스텀 미들웨어(MyCustomMiddleware)와 이를 등록하는 확장 메서드"
date: 2025-07-21 20:11 +0900
author: hyesung
description: UseMyCustomMiddleware를 별도 파일로 분리하면, Program.cs가 매우 간결해지고 유지보수가 쉬워진다.
---
## ASP.NET Core 커스텀 미들웨어 및 확장 메서드 정리

ASP.NET Core 애플리케이션에서 특정 요청 처리 로직을 삽입하려면 **미들웨어(Middleware)**를 사용한다. 이 강의에서는 커스텀 미들웨어를 만들고, 이를 더 간결하게 요청 파이프라인에 등록하기 위한 **확장 메서드(Extension Method)**를 구현하는 방법을 다룬다.

---

### 1. 커스텀 미들웨어 정의

먼저 요청 처리 흐름에 개입할 커스텀 미들웨어 클래스를 정의한다.

```C#
using System.Net.NetworkInformation;
using System.Runtime.CompilerServices;
using Microsoft.AspNetCore.Http; // HttpContext, RequestDelegate를 위해 필요

namespace MiddlewareExample.CustomMiddleware
{
    public class MyCustomMilddleware : IMiddleware // IMiddleware 인터페이스를 구현한다.
    {
        // 이 미들웨어 클래스는 DI 컨테이너에 등록해야 한다. (아래 2.1 참고)
        public async Task InvokeAsync(HttpContext context, RequestDelegate next)
        {
            // [요청 처리 전 로직]
            await context.Response.WriteAsync("Custom Middleware Executed!\n");

            // 다음 미들웨어 호출
            await next(context);

            // [다음 미들웨어 완료 후 로직]
            // next 미들웨어가 종료되면 실행되는 부분이다.
            await context.Response.WriteAsync("Custom Middleware Completed!\n");
        }
    }
}
```

- **`IMiddleware` 인터페이스**: 커스텀 미들웨어가 구현해야 하는 인터페이스이다. 이를 통해 DI 컨테이너에 미들웨어를 등록하고 관리할 수 있다.
- **`InvokeAsync` 메서드**: 실제 미들웨어 로직이 구현되는 비동기 메서드이다.
    - `HttpContext context`: 현재 HTTP 요청 및 응답에 대한 정보를 담고 있다.
    - `RequestDelegate next`: 요청 파이프라인의 다음 미들웨어를 나타내는 델리게이트이다. `await next(context);`를 호출하여 다음 미들웨어로 제어를 넘긴다.
    - `await next(context);` **이전**: 요청이 다음 미들웨어로 전달되기 전에 실행되는 로직이다.
    - `await next(context);` **이후**: 다음 미들웨어의 처리가 완료되고 제어가 다시 돌아왔을 때 실행되는 로직이다.

---
### 2. 커스텀 미들웨어 등록 및 확장 메서드 활용

커스텀 미들웨어를 애플리케이션의 요청 파이프라인에 추가하는 방법과, 이를 더 깔끔하게 처리하기 위한 확장 메서드 생성에 대해 알아본다.

#### 2.1. DI 컨테이너에 미들웨어 등록

`MyCustomMilddleware`는 `IMiddleware`를 구현하므로, 의존성 주입(DI) 컨테이너에 등록해야 한다. 일반적으로 `Program.cs` 파일에서 `AddTransient`로 등록한다.

```C#
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddTransient<MyCustomMilddleware>(); // MyCustomMilddleware를 Transient 서비스로 등록한다.
```

- **`AddTransient<T>`**: 요청이 발생할 때마다 새로운 인스턴스를 생성하는 방식으로, 미들웨어에 적합하다.

#### 2.2. 요청 파이프라인에 미들웨어 추가 (확장 메서드 사용 전)

미들웨어를 DI 컨테이너에 등록한 후에는 `app.UseMiddleware<T>()` 메서드를 사용하여 요청 파이프라인에 추가할 수 있다.

```C#
// Program.cs (확장 메서드 사용 전)
var app = builder.Build();
// app.UseMiddleware<MyCustomMilddleware>(); // 이렇게 직접 호출할 수 있다.
```
#### 2.3. 커스텀 미들웨어용 확장 메서드 생성
`app.UseMiddleware<MyCustomMilddleware>()`와 같이 미들웨어를 직접 호출하는 대신, 미리 정의된 미들웨어(`UseAuthentication()`, `UseAuthorization()` 등)처럼 간단한 메서드 호출로 커스텀 미들웨어를 추가할 수 있도록 **확장 메서드**를 만든다.

```C#
// MiddlewareExample.CustomMiddleware 네임스페이스 내 (MyCustomMilddleware.cs 파일 또는 별도 파일)
public static class CustomMiddlewareExtension // 확장 메서드를 담을 static 클래스
{
    // 확장 메서드로 커스텀 미들웨어를 등록하는 메서드
    // 'this IApplicationBuilder app' 부분이 핵심이다.
    // 이 'app' 인수는 Program.cs의 'app' 객체(WebApplication 타입)와 동일하다.
    // WebApplication은 IApplicationBuilder의 하위 클래스이므로, IApplicationBuilder에 확장 메서드를 추가하면 WebApplication 객체에서도 사용할 수 있다.
    public static IApplicationBuilder UseMyCustomMiddleware(this IApplicationBuilder app)
    {
        // UseMiddleware<T> 메서드는 IApplicationBuilder 객체를 반환하므로 그대로 반환한다.
        return app.UseMiddleware<MyCustomMilddleware>();
    }
}
```

- **`static class`**: 확장 메서드를 포함하는 클래스는 반드시 `static`이어야 한다.
- **`static method`**: 확장 메서드 자체도 반드시 `static`이어야 한다.
- **`this IApplicationBuilder app`**: 확장 메서드의 첫 번째 매개변수에 `this` 키워드를 붙이고, 확장하려는 타입(`IApplicationBuilder`)을 지정한다. 이렇게 하면 해당 타입의 객체(`app`)에서 이 메서드를 마치 자신의 멤버 메서드처럼 호출할 수 있게 된다.
- **메서드 이름 컨벤션**: ASP.NET Core에서는 미들웨어 확장 메서드 이름에 **`Use`** 접두사를 붙이는 것이 일반적인 컨벤션이다. (예: `UseRouting`, `UseEndpoints`, `UseAuthentication` 등)

#### 2.4. 요청 파이프라인에 미들웨어 추가 (확장 메서드 사용)

이제 `Program.cs`에서 생성한 확장 메서드를 사용해 커스텀 미들웨어를 요청 파이프라인에 추가할 수 있다.

```C#
// Program.cs (확장 메서드 사용 후)
using MiddlewareExample.CustomMiddleware; // 확장 메서드를 사용하기 위해 네임스페이스를 참조한다.

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddTransient<MyCustomMilddleware>();

var app = builder.Build();

app.UseMyCustomMiddleware(); // 직접 작성한 확장 메서드를 호출한다. (app.UseMiddleware<MyCustomMilddleware>()를 대체)

// 다른 미들웨어들...
app.Run(async (HttpContext context) => // middleware 2 (예시)
{
    await context.Response.WriteAsync("hello 2\n");
});

app.Run(async (HttpContext context) => // middleware 3 (예시)
{
    await context.Response.WriteAsync("hello 3\n"); // 기존 "hello 2" -> "hello 3"으로 수정
});

app.Run(); // 애플리케이션 실행
```

---
### 3. 미들웨어 실행 흐름 시연

미들웨어는 요청 파이프라인에 등록된 순서대로 실행되며, `next(context)` 호출을 통해 다음 미들웨어로 제어를 넘기고, 다음 미들웨어의 처리가 완료되면 다시 원래 미들웨어로 돌아와 나머지 로직을 실행한다.

예시 코드의 실행 흐름은 다음과 같다.

1. **요청 발생**
2. **`app.UseMyCustomMiddleware()`** (커스텀 미들웨어 실행):
    - "Custom Middleware Executed!" 출력
    - `next(context)` 호출 (다음 미들웨어로 이동)
3. **`app.Run(async (HttpContext context) => { await context.Response.WriteAsync("hello 2\n"); })`** (미들웨어 2 실행):
    - "hello 2" 출력
    - **주의**: `app.Run`은 파이프라인의 끝을 나타내므로, 보통 파이프라인의 마지막에 사용된다. 이 예시처럼 `Run`을 두 번 사용하면 첫 번째 `Run` 미들웨어에서 요청 처리가 끝나고 두 번째 `Run` 미들웨어는 실행되지 않는다. 일반적인 미들웨어 체인은 `Use`를 사용하고 마지막에 `Run` 또는 최종 엔드포인트 미들웨어를 둔다.
    - (만약 `app.Use(...)` 형태였다면, 미들웨어 2가 완료된 후 다시 커스텀 미들웨어로 제어가 돌아온다.)
4. **`app.UseMyCustomMiddleware()`로 제어 복귀**:
    - "Custom Middleware Completed!" 출력
    - 커스텀 미들웨어 처리 완료.

---
### 4. 요약

- **커스텀 미들웨어**: `IMiddleware` 인터페이스를 구현하고 `InvokeAsync` 메서드에 요청 처리 로직을 작성한다. `next(context)`를 기준으로 **전/후** 로직을 분리할 수 있다.
- **확장 메서드**: 미들웨어를 요청 파이프라인에 등록할 때 **코드의 가독성**을 높이고 **재사용성**을 확보하기 위해 사용한다. `static` 클래스 내부에 `this` 키워드를 사용한 `static` 메서드로 정의하며, 주로 `Use` 접두사를 붙여 명명한다.
- **DI 컨테이너**: `IMiddleware`를 구현한 커스텀 미들웨어는 `builder.Services.AddTransient<MyCustomMilddleware>()`와 같이 DI 컨테이너에 등록해야 한다.