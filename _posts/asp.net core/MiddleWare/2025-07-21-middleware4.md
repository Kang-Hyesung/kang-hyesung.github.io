---
title: "[4] ASP.NET Core 커스텀 미들웨어 및 확장 메서드 정리"
date: 2025-07-21 20:11 +0900
author: hyesung
---
ASP.NET Core 애플리케이션에서 요청 처리 파이프라인에 사용자 정의 로직을 삽입하려면 **미들웨어(Middleware)**를 사용한다. 이 글에서는 커스텀 미들웨어를 정의하고, 이를 파이프라인에 더 간결하게 추가하기 위한 **확장 메서드(Extension Method)**를 구현하는 방법을 설명한다.

---

### 1. 커스텀 미들웨어 정의

테스트테스트

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

안녕하세요\! 제공해주신 C\# 커스텀 미들웨어(Middleware) 구현에 대한 스크립트를 바탕으로, 가독성을 높이고 핵심 개념을 명확히 하는 깃허브(GitHub) 기술 블로그 스타일의 글로 재구성해 보았습니다.

특히 C\#의 미들웨어 개념이 Java Spring 생태계에서는 어떻게 구현되는지 비교 분석하는 내용을 추가하여 C\# 개발자분들이 Spring 프레임워크를 이해하는 데 도움이 되도록 구성했습니다.

-----

알겠습니다. 요청하신 대로 모든 문장의 끝을 `~다.`로 통일하여, 보다 전문적이고 일관된 톤의 기술 문서 스타일로 수정했습니다.

-----

## **[C\#] ASP.NET Core 미들웨어(Middleware) 완벽 가이드: C\#부터 Java Spring까지**

테스트123

### **도입**

웹 애플리케이션을 개발할 때, 모든 요청에 공통적으로 적용해야 하는 기능들이 있다. 예를 들어, 모든 요청에 대한 로그(Log)를 남기거나, 사용자의 인증(Authentication) 상태를 확인하고, 특정 IP의 접근을 차단하는 등의 기능이 그렇다. 이러한 \*\*횡단 관심사(Cross-Cutting Concerns)\*\*를 효율적으로 처리하기 위해 ASP.NET Core는 **미들웨어(Middleware)** 라는 강력한 파이프라인(Pipeline) 모델을 제공한다.

이번 포스트에서는 ASP.NET Core에서 커스텀 미들웨어를 직접 정의하고, 요청 파이프라인에 등록하여 사용하는 전체 과정을 단계별로 알아본다. 또한, 이러한 개념이 **Java Spring 환경**에서는 어떤 기술(Filter, Interceptor)로 구현되는지 비교하여 폭넓은 이해를 돕고자 한다.

### **1. 커스텀 미들웨어의 정의 (`IMiddleware`)**

가장 먼저 요청 처리 흐름에 개입할 우리만의 미들웨어 클래스를 정의해야 한다. 이 클래스는 반드시 `IMiddleware` 인터페이스를 구현해야 프레임워크가 미들웨어로 인식할 수 있다.

```csharp
// MiddlewareExample/CustomMiddleware/MyCustomMiddleware.cs

using Microsoft.AspNetCore.Http;
using System.Threading.Tasks;

namespace MiddlewareExample.CustomMiddleware
{
    public class MyCustomMiddleware : IMiddleware // IMiddleware 인터페이스를 구현한다.
    {
        // InvokeAsync 메서드는 미들웨어의 핵심 로직을 담는다.
        public async Task InvokeAsync(HttpContext context, RequestDelegate next)
        {
            // [1. 요청 처리 전 로직]
            // 다음 미들웨어가 실행되기 전에 수행할 코드를 작성한다.
            await context.Response.WriteAsync("Custom Middleware Executed!\n");

            // [2. 다음 미dleware 호출]
            // next 델리게이트를 호출하여 요청을 파이프라인의 다음 미들웨어로 전달한다.
            // 이 호출을 기준으로 코드가 나뉜다.
            await next(context);

            // [3. 다음 미들웨어 완료 후 로직]
            // 'next'로 호출된 모든 미들웨어의 실행이 끝난 후, 제어권이 다시 돌아와서 실행된다.
            await context.Response.WriteAsync("Custom Middleware Completed!\n");
        }
    }
}
```

  - **`IMiddleware` 인터페이스**: 프레임워크가 미들웨어를 인식하고 **의존성 주입(Dependency Injection, DI)** 컨테이너를 통해 생명주기를 관리할 수 있도록 하는 핵심 인터페이스다.
  - **`InvokeAsync(HttpContext context, RequestDelegate next)` 메서드**:
      - `HttpContext context`: 현재 HTTP 요청과 응답에 대한 모든 정보(헤더, 바디, 쿼리스트링 등)를 담고 있는 객체다.
      - `RequestDelegate next`: 파이프라인의 **다음 미들웨어를 가리키는 대리자(Delegate)** 이다. `await next(context);`를 호출하면 제어 흐름이 다음 미들웨어로 넘어간다. 이 호출이 없다면 파이프라인은 여기서 멈추게 된다(단락, Short-circuit).

-----

### **2. 미들웨어 등록과 확장 메서드의 마법**

미들웨어를 정의했다면, 이제 애플리케이션이 이를 사용할 수 있도록 등록해야 한다.

#### **2.1. DI 컨테이너에 서비스로 등록하기**

`IMiddleware`를 구현한 클래스는 생성자의 매개변수를 통해 다른 서비스를 주입받을 수 있다. 이를 위해 반드시 DI 컨테이너에 등록되어야 한다. `Program.cs` 파일에서 다음과 같이 등록할 수 있다.

```csharp
// Program.cs
using MiddlewareExample.CustomMiddleware;

var builder = WebApplication.CreateBuilder(args);

// MyCustomMiddleware를 Transient 수명 주기로 DI 컨테이너에 등록한다.
// Transient는 요청마다 새로운 인스턴스를 생성하므로 미들웨어에 적합하다.
builder.Services.AddTransient<MyCustomMiddleware>();

// ...
```

#### **2.2. 가독성을 높이는 확장 메서드 (`UseMyCustomMiddleware`)**

ASP.NET Core는 `app.UseAuthentication()`, `app.UseAuthorization()`처럼 직관적인 메서드로 미들웨어를 등록한다. 우리도 이처럼 간결한 코드를 위해 \*\*확장 메서드(Extension Method)\*\*를 만드는 것이 좋다.

```csharp
// MiddlewareExample/CustomMiddleware/CustomMiddlewareExtension.cs

using Microsoft.AspNetCore.Builder;

namespace MiddlewareExample.CustomMiddleware
{
    // 확장 메서드는 반드시 static 클래스 내부에 static 메서드로 정의해야 한다.
    public static class CustomMiddlewareExtension
    {
        // 'this IApplicationBuilder app' 구문을 통해 IApplicationBuilder 타입을 확장한다.
        // 'Use' 접두사는 ASP.NET Core의 미들웨어 명명 규칙(Convention)이다.
        public static IApplicationBuilder UseMyCustomMiddleware(this IApplicationBuilder app)
        {
            // UseMiddleware<T>를 호출하여 DI 컨테이너에 등록된 미들웨어를 파이프라인에 추가한다.
            return app.UseMiddleware<MyCustomMiddleware>();
        }
    }
}
```

  - **`this IApplicationBuilder app`**: `IApplicationBuilder` 타입의 객체에서 `UseMyCustomMiddleware()` 메서드를 마치 원래 있던 멤버 메서드처럼 호출할 수 있게 해주는 핵심 구문이다.
  - **`Use` 접두사**: ASP.NET Core의 미들웨어 확장 메서드는 관례적으로 `Use`로 시작한다. 이 규칙을 따르면 코드의 일관성과 가독성이 크게 향상된다.

-----

### **3. 요청 파이프라인 구성 및 실행 흐름 분석**

이제 `Program.cs`에서 우리가 만든 확장 메서드를 사용하여 전체 요청 파이프라인을 구성하고, 그 실행 흐름을 깊이 있게 살펴본다.

```csharp
// Program.cs
using MiddlewareExample.CustomMiddleware; // 확장 메서드를 사용하기 위해 네임스페이스를 참조한다.

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddTransient<MyCustomMiddleware>();

var app = builder.Build();

// 1번 미들웨어 (app.Use 람다식)
app.Use(async (context, next) =>
{
    await context.Response.WriteAsync("Middleware 1 - Start\n");
    await next(context); // 2번 미들웨어(MyCustomMiddleware) 호출
    await context.Response.WriteAsync("Middleware 1 - End\n");
});

// 2번 미들웨어 (커스텀 확장 메서드)
app.UseMyCustomMiddleware();

// 3번 미들웨어 (app.Run 터미널)
app.Run(async (context) =>
{
    await context.Response.WriteAsync("Terminal Middleware (Last)\n");
});

app.Run();
```

#### **실행 흐름 시연**

위 코드는 요청이 들어왔을 때 다음과 같은 "양파 껍질(Onion-like)" 구조로 동작한다.

1.  **요청 시작 → `Middleware 1` 진입**

      - `"Middleware 1 - Start"`가 응답에 기록된다.
      - `await next(context)`를 만나 제어권을 `MyCustomMiddleware`로 넘긴다.

2.  **`MyCustomMiddleware` 진입**

      - `InvokeAsync` 메서드의 `next` 호출 전 로직이 실행되어 `"Custom Middleware Executed!"`가 기록된다.
      - 다시 `await next(context)`를 만나 제어권을 터미널 미들웨어로 넘긴다.

3.  **`Terminal Middleware` 실행**

      - `app.Run`으로 등록된 마지막 미들웨어가 실행되어 `"Terminal Middleware (Last)"`가 기록된다.
      - `app.Run`은 `next`를 호출하지 않으므로 여기서 파이프라인은 "반환"을 시작한다.

4.  **`MyCustomMiddleware`로 제어 복귀**

      - `await next(context)` 호출이 끝났으므로, 그 다음 라인인 `"Custom Middleware Completed!"`가 기록된다. `MyCustomMiddleware`의 역할이 끝난다.

5.  **`Middleware 1`로 제어 복귀**

      - `MyCustomMiddleware`가 끝났으므로, `Middleware 1`의 `await next(context)` 호출이 끝난 것으로 간주된다.
      - 그 다음 라인인 `"Middleware 1 - End"`가 기록된다. `Middleware 1`의 역할도 끝난다.

#### **최종 응답 결과**

클라이언트가 받는 최종 응답 본문은 아래와 같다.

```
Middleware 1 - Start
Custom Middleware Executed!
Terminal Middleware (Last)
Custom Middleware Completed!
Middleware 1 - End
```

-----

### **Java Spring에서는? (Filter와 Interceptor)**

ASP.NET Core의 미들웨어와 같은 **횡단 관심사 처리** 개념은 Java Spring 생태계에도 존재한다. Spring에서는 주로 \*\*서블릿 필터(Servlet Filter)\*\*와 \*\*핸들러 인터셉터(Handler Interceptor)\*\*라는 두 가지 기술을 사용한다.

#### **1. 서블릿 필터 (Servlet Filter)**

  - **개념**: `Filter`는 Spring 프레임워크의 가장 바깥 단, 즉 **서블릿 컨테이너(Servlet Container) 레벨**에서 동작한다. 요청이 Spring의 핵심인 `DispatcherServlet`에 도달하기 **전**에 가로챈다.
  - **C\# 매핑**: ASP.NET Core의 **미들웨어와 가장 유사한 개념**이다.
  - **주요 용도**: 인코딩 변환, 보안(예: Spring Security), 전체 요청/응답 로깅 등 애플리케이션 전반에 걸친 저수준 처리에 적합하다.

**Java 예시 코드:**

```java
// LogFilter.java
import javax.servlet.*;
import java.io.IOException;

// @Component // Spring Bean으로 등록
public class LogFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("--- Filter: Before Request ---"); // next.invoke() 이전 로직
        chain.doFilter(request, response); // 다음 필터 또는 서블릿으로 요청 전달
        System.out.println("--- Filter: After Response ---"); // next.invoke() 이후 로직
    }
}
```

#### **2. 핸들러 인터셉터 (Handler Interceptor)**

  - **개념**: `Interceptor`는 `DispatcherServlet`이 요청을 받은 **후**, 컨트롤러의 특정 핸들러 메서드(`@GetMapping`, `@PostMapping` 등)를 실행하기 **전과 후**에 개입한다.
  - **C\# 매핑**: 미들웨어보다 더 **애플리케이션(MVC) 계층에 특화된** 버전이라고 볼 수 있다.
  - **주요 용도**: 특정 URL 패턴에만 적용되는 인증/인가 체크, 컨트롤러로 넘어가는 데이터의 전처리, API 성능 측정 등 비즈니스 로직과 더 가까운 작업에 사용된다. Spring 컨텍스트에 접근하기 용이하다.

**Java 예시 코드:**

```java
// LogInterceptor.java
import org.springframework.web.servlet.HandlerInterceptor;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

// @Component // Spring Bean으로 등록
public class LogInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("--- Interceptor: Before Handler ---"); // 컨트롤러 실행 전
        return true; // true를 반환해야 컨트롤러가 실행됨
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("--- Interceptor: After Handler ---"); // 컨트롤러 실행 후, 뷰 렌더링 전
    }
}
```

#### **핵심 비교**

| 구분             | **ASP.NET Core Middleware** | **Spring Filter** | **Spring Interceptor** |
| ---------------- | ------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------ |
| **작동 레벨** | HTTP 파이프라인 (프레임워크 최상단)                          | 서블릿 컨테이너 (Spring 프레임워크 진입 전)              | Spring MVC 컨텍스트 (DispatcherServlet 내부)                 |
| **역할** | 요청/응답에 대한 광범위하고 공통적인 처리                      | 인코딩, 보안 등 저수준의 전역적 처리                     | 컨트롤러 핸들러와 연계된 세밀한 로직 처리                      |
| **컨텍스트 접근** | `HttpContext` (HTTP 요청/응답의 모든 정보)                   | `ServletRequest`, `ServletResponse` (HTTP 정보)          | `HandlerMethod` 등 Spring MVC의 내부 정보에 접근 용이        |
| **가장 비슷한 개념** | \*\*`Filter`\*\*와 동작 방식 및 역할이 매우 유사하다.          | \*\*`Middleware`\*\*와 거의 1:1로 대응된다.                  | 특정 경로/조건에만 적용되는 조건부 미들웨어와 유사하다.      |

### **결론**

ASP.NET Core의 미들웨어는 요청 파이프라인을 통해 애플리케이션의 공통 기능을 모듈화하는 강력하고 유연한 방법이다. `IMiddleware` 구현, DI 컨테이너 등록, 그리고 확장 메서드를 통한 간결한 사용법은 체계적이고 유지보수하기 좋은 코드를 작성하는 데 큰 도움이 된다.

또한, Spring의 `Filter`와 `Interceptor` 개념을 통해 알 수 있듯, **"요청의 흐름 중간에 개입하여 공통 로직을 처리한다"**는 아이디어는 현대 웹 프레임워크의 핵심적인 디자인 패턴 중 하나다. 두 기술 스택의 유사점과 차이점을 이해한다면, 어떤 환경에서도 견고한 웹 애플리케이션을 설계하는 역량을 기를 수 있다.