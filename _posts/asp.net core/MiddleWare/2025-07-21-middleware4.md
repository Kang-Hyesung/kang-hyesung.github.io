---
title: "[4] ASP.NET Core 커스텀 미들웨어 및 확장 메서드 정리"
date: 2025-07-21 20:11 +0900
author: hyesung
---
ASP.NET Core 애플리케이션에서 요청 처리 파이프라인에 사용자 정의 로직을 삽입하려면 **미들웨어(Middleware)**를 사용한다. 이 글에서는 커스텀 미들웨어를 정의하고, 이를 파이프라인에 더 간결하게 추가하기 위한 **확장 메서드(Extension Method)**를 구현하는 방법을 설명한다.

---

### 1. 커스텀 미들웨어 정의

요청 처리 흐름에 개입할 커스텀 미들웨어 클래스를 먼저 정의한다. 이 클래스는 `IMiddleware` 인터페이스를 구현해야 한다.

```csharp
using Microsoft.AspNetCore.Http; // HttpContext, RequestDelegate를 위해 필요

namespace MiddlewareExample.CustomMiddleware
{
    public class MyCustomMilddleware : IMiddleware // IMiddleware 인터페이스를 구현한다.
    {
        // 이 미들웨어 클래스는 DI 컨테이너에 등록되어야 한다. (아래 2.1 참고)
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

- **`IMiddleware` 인터페이스**: 커스텀 미들웨어가 구현해야 하는 핵심 인터페이스이다. 이를 통해 프레임워크가 미들웨어를 인식하고 DI 컨테이너를 통해 관리할 수 있다.
- **`InvokeAsync` 메서드**: 실제 미들웨어의 비동기 로직이 구현되는 곳이다.
    - `HttpContext context`: 현재 HTTP 요청 및 응답에 대한 모든 정보를 담고 있다.
    - `RequestDelegate next`: 요청 파이프라인의 다음 미들웨어를 나타내는 델리게이트이다. `await next(context);`를 호출하여 요청을 다음 미들웨어로 전달한다.

---
### 2. 커스텀 미들웨어 등록 및 확장 메서드 활용

커스텀 미들웨어를 애플리케이션의 요청 파이프라인에 추가하는 방법과, 이를 더욱 간결하게 만드는 확장 메서드 구현에 대해 설명한다.
#### 2.1. DI 컨테이너에 미들웨어 등록

`MyCustomMilddleware`는 `IMiddleware`를 구현하므로, 의존성 주입(DI) 컨테이너에 등록해야 프레임워크가 이를 관리하고 인스턴스화할 수 있다. 일반적으로 `Program.cs` 파일에서 `AddTransient`로 등록한다.

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddTransient<MyCustomMilddleware>(); // MyCustomMilddleware를 Transient 서비스로 등록한다.
```

- **`AddTransient<T>`**: 요청이 발생할 때마다 새로운 미들웨어 인스턴스를 생성하는 방식으로, 미들웨어에 적합한 수명 주기이다.

#### 2.2. 커스텀 미들웨어용 확장 메서드 생성

ASP.NET Core의 내장 미들웨어들(`UseAuthentication()`, `UseAuthorization()` 등)처럼 간단한 메서드 호출로 커스텀 미들웨어를 추가할 수 있도록 **확장 메서드**를 만든다. 이는 코드의 가독성을 높이고, 미들웨어 등록을 표준화하는 데 도움을 준다.

```csharp
// MiddlewareExample.CustomMiddleware 네임스페이스 내 (MyCustomMilddleware.cs 파일 또는 별도 파일)
public static class CustomMiddlewareExtension // 확장 메서드를 담을 static 클래스이다.
{
    // 확장 메서드로 커스텀 미들웨어를 등록하는 메서드
    // 'this IApplicationBuilder app' 부분이 핵심이다.
    // 이 'app' 인수는 Program.cs의 'app' 객체(WebApplication 타입)와 동일하다.
    // WebApplication 클래스는 IApplicationBuilder의 하위 클래스이므로,
    // IApplicationBuilder에 확장 메서드를 추가하면 WebApplication 객체에서도 해당 메서드를 사용할 수 있다.
    public static IApplicationBuilder UseMyCustomMiddleware(this IApplicationBuilder app)
    {
        // UseMiddleware<T> 메서드는 IApplicationBuilder 객체를 반환하므로 이를 그대로 반환한다.
        return app.UseMiddleware<MyCustomMilddleware>();
    }
}
```

- **`static class`**: 확장 메서드를 포함하는 클래스는 반드시 `static`이어야 한다.
- **`static method`**: 확장 메서드 자체도 반드시 `static`이어야 한다.
- **`this IApplicationBuilder app`**: 확장하려는 타입(`IApplicationBuilder`) 앞에 `this` 키워드를 붙인다. 이렇게 하면 해당 타입의 객체(`app`)에서 이 메서드를 마치 자신의 멤버 메서드처럼 호출할 수 있게 된다.
- **메서드 이름 컨벤션**: ASP.NET Core에서는 미들웨어 확장 메서드 이름에 **`Use`** 접두사를 붙이는 것이 일반적인 컨벤션이다. (예: `UseRouting`, `UseEndpoints`, `UseAuthentication` 등)

---
### 3. 요청 파이프라인 구성 및 미들웨어 실행 흐름

이제 `Program.cs`에서 확장 메서드를 사용하여 커스텀 미들웨어를 포함한 요청 파이프라인을 구성하고, 그 실행 흐름을 분석한다.

```csharp
// Program.cs
using MiddlewareExample.CustomMiddleware; // 확장 메서드를 사용하기 위해 네임스페이스를 참조한다.

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddTransient<MyCustomMilddleware>();

var app = builder.Build();

app.Use(async (HttpContext context, RequestDelegate next) => // middleware 1 (app.Use()로 정의)
{
    await context.Response.WriteAsync("hello 1\n"); // 첫 번째 미들웨어 진입 시 메시지 출력
    await next(context); // 다음 미들웨어(MyCustomMilddleware) 호출
    // MyCustomMilddleware와 Middleware 2의 처리가 완료된 후 제어가 돌아오는 지점
    await context.Response.WriteAsync("Middleware 1 Completed!\n"); // Middleware 1 종료 메시지
});

app.UseMyCustomMiddleware(); // 확장 메서드를 통해 CustomMiddleware 호출 (파이프라인의 두 번째)

app.Run(async (HttpContext context) => // middleware 2 (터미널 미들웨어, 파이프라인의 세 번째이자 마지막)
{
    await context.Response.WriteAsync("hello 2\n"); // 파이프라인의 끝에서 메시지 출력
});

app.Run(); // 애플리케이션 실행
```

---
### 4. 미들웨어 실행 흐름 시연

미들웨어는 요청 파이프라인에 등록된 순서대로 실행되며, `next(context)` 호출을 통해 다음 미들웨어로 제어를 넘기고, 다음 미들웨어의 처리가 완료되면 다시 원래 미들웨어로 돌아와 나머지 로직을 실행한다.
예시 코드의 실행 흐름은 다음과 같다.
1. **요청 발생**: 클라이언트로부터 HTTP 요청이 들어온다.
2. **`app.Use(...)` (Middleware 1) 실행**:
    - 파이프라인에 **첫 번째**로 등록된 Middleware 1이 실행된다.
    - `await context.Response.WriteAsync("hello 1\n");`이 실행되어 응답에 "hello 1"이 추가된다.
    - `await next(context);`가 호출된다. Middleware 1은 다음 미들웨어의 처리가 완료될 때까지 잠시 대기 상태가 된다.
3. **`app.UseMyCustomMiddleware()` (MyCustomMilddleware) 실행**:
    - Middleware 1에서 `next(context)`를 호출했으므로, 파이프라인에 **두 번째**로 등록된 `MyCustomMilddleware`가 실행된다.
    - `await context.Response.WriteAsync("Custom Middleware Executed!\n");`이 실행되어 응답에 "Custom Middleware Executed!"가 추가된다.
    - `await next(context);`가 호출된다. `MyCustomMilddleware` 또한 다음 미들웨어(Middleware 2)의 처리가 완료될 때까지 대기 상태가 된다.
4. **`app.Run(...)` (Middleware 2) 실행**:
    - `MyCustomMilddleware`에서 `next(context)`를 호출했으므로, 파이프라인에 **세 번째**이자 **마지막**으로 등록된 `app.Run()` 미들웨어(Middleware 2)가 실행된다.
    - `await context.Response.WriteAsync("hello 2\n");`가 실행되어 응답에 "hello 2"가 추가된다.
    - **이 미들웨어는 `app.Run()`이므로 파이프라인을 종료한다.** `next()`를 호출하지 않으며, 요청 처리를 여기서 완료하고 응답을 클라이언트에 보낸다.
5. **`MyCustomMilddleware`로 제어 복귀**:
    - Middleware 2의 처리가 완료되었으므로, 제어는 Middleware 2를 호출했던 **`MyCustomMilddleware`의 `await next(context);` 이후 지점으로 돌아온다.**
    - `await context.Response.WriteAsync("Custom Middleware Completed!\n");`이 실행되어 응답에 "Custom Middleware Completed!"가 추가된다.
6. **`Middleware 1`로 제어 복귀**:
    - `MyCustomMilddleware`의 처리가 완료되었으므로, 제어는 `MyCustomMilddleware`를 호출했던 **Middleware 1의 `await next(context);` 이후 지점으로 돌아온다.**
    - `await context.Response.WriteAsync("Middleware 1 Completed!\n");`이 실행되어 응답에 "Middleware 1 Completed!"가 추가된다.

**최종 응답 결과**: 브라우저에는 다음과 같은 순서로 메시지가 출력된다.

```
hello 1
Custom Middleware Executed!
hello 2
Custom Middleware Completed!
Middleware 1 Completed!
```

---

### 5. 핵심 요약
- **미들웨어 파이프라인 순서**: 미들웨어는 `Program.cs` 파일에 등록된 순서대로 실행된다.
- **`app.Use()` 미들웨어**: `RequestDelegate next`를 매개변수로 받고 `await next(context);`를 호출하여 다음 미들웨어로 제어를 넘긴다. 다음 미들웨어의 처리가 완료되면 자신에게 **제어가 다시 돌아와서** `next(context)` 이후의 코드를 실행한다. 이는 미들웨어 체인이 스택처럼 동작하는 핵심 원리이다.
- **`app.Run()` 미들웨어**: `RequestDelegate next`를 매개변수로 받지 않으며 `next(context)`를 호출하지 않는다. 이 미들웨어가 실행되면 **요청 파이프라인이 종료**되고, 응답이 시작된다. 이 미들웨어 뒤에 오는 다른 미들웨어는 실행되지 않으며, 자신을 호출한 이전 미들웨어로 제어가 돌아오지 않는다. 따라서 `app.Run()`은 항상 파이프라인의 **마지막**에 위치해야 한다.
- **확장 메서드**: 미들웨어를 요청 파이프라인에 등록할 때 **코드의 가독성**을 높이고 **재사용성**을 확보하기 위해 사용한다. `static` 클래스 내부에 `this` 키워드를 사용한 `static` 메서드로 정의하며, 주로 `Use` 접두사를 붙여 명명하는 것이 관례이다.
- **DI 컨테이너**: `IMiddleware`를 구현한 커스텀 미들웨어는 `builder.Services.AddTransient<MyCustomMilddleware>()`와 같이 DI 컨테이너에 등록한다.