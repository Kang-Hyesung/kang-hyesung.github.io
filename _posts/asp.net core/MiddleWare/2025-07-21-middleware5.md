---
title: "[5] 인터페이스 없는 미들웨어 생성 및 활용"
date: 2025-07-21 23:03 +0900
author: hyesung
---
ASP.NET Core에서는 `IMiddleware` 인터페이스를 상속하지 않고도 **규약(convention)**에 따라 사용자 지정 미들웨어를 생성할 수 있다. ASP.NET Core 6부터 이 방식이 권장되며, 이는 미들웨어 클래스의 유연성을 높인다.

---

### 1. `IMiddleware` 인터페이스 없는 미들웨어 정의

일반 C# 클래스를 미들웨어로 사용하려면 다음과 같은 특정 규약을 따라야 한다.


```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;
using System.Threading.Tasks;

namespace MiddlewareExample
{
    // Microsoft.AspNetCore.Http.Abstractions 패키지 설치 필요
    public class HelloCustomMiddleware
    {
        private readonly RequestDelegate _next; // 다음 미들웨어를 참조할 필드

        // 생성자를 통해 다음 미들웨어(RequestDelegate)를 주입받는다.
        public HelloCustomMiddleware(RequestDelegate next)
        {
            _next = next;
        }

        // Invoke 또는 InvokeAsync 메서드를 정의하고 HttpContext를 매개변수로 받는다.
        public async Task Invoke(HttpContext httpContext)
        {
            // [요청 처리 전 로직 (Before Logic)]
            // 쿼리 문자열에 'firstname'과 'lastname' 키가 모두 있는지 확인한다.
            if (httpContext.Request.Query.ContainsKey("firstname") && httpContext.Request.Query.ContainsKey("lastname"))
            {
                var firstName = httpContext.Request.Query["firstname"];
                var lastName = httpContext.Request.Query["lastname"];

                // 두 값이 모두 있으면 전체 이름을 구성하여 JSON 형식으로 응답에 기록한다.
                await httpContext.Response.WriteAsJsonAsync(firstName + " " + lastName);
            }

            // 다음 미들웨어 호출
            await _next(httpContext);

            // [요청 처리 후 로직 (After Logic)]
            // 다음 미들웨어의 처리가 완료된 후 실행된다.
        }
    }

    // 미들웨어를 HTTP 요청 파이프라인에 추가하기 위한 확장 메서드
    public static class HelloCustomMiddlewareExtensions
    {
        public static IApplicationBuilder UseHelloCustomMiddleware(this IApplicationBuilder builder)
        {
            // UseMiddleware<T> 메서드를 사용하여 미들웨어를 등록한다.
            return builder.UseMiddleware<HelloCustomMiddleware>();
        }
    }
}
```

#### 주요 특징

- **생성자 주입**: `RequestDelegate _next` 필드를 선언하고, 생성자를 통해 `RequestDelegate next`를 주입받는다. ASP.NET Core는 해당 미들웨어의 인스턴스를 생성할 때 자동으로 다음 미들웨어 체인을 `next` 매개변수로 전달한다.
- **`Invoke` 또는 `InvokeAsync` 메서드**: HTTP 요청을 처리하는 핵심 메서드이다. 반드시 `HttpContext`를 첫 번째 매개변수로 받아야 한다. 비동기 작업이 포함될 경우 `InvokeAsync`로 정의한다.
- **`_next(httpContext)` 호출**: 이전 `IMiddleware` 방식과 동일하게, `_next(httpContext)`를 호출하여 파이프라인의 다음 미들웨어로 제어를 넘긴다. 이 호출 전후로 'Before Logic'과 'After Logic'을 구현할 수 있다.
- **DI 컨테이너 등록 불필요**: 이 방식의 미들웨어는 `IMiddleware` 인터페이스를 구현하지 않으므로, `builder.Services.AddTransient<HelloCustomMiddleware>()`와 같이 DI 컨테이너에 직접 등록할 필요가 없다. `app.UseMiddleware<HelloCustomMiddleware>()`가 내부적으로 미들웨어를 DI 컨테이너에 등록하고 관리한다.

---

### 2. 요청 파이프라인에 미들웨어 추가

이 방식의 미들웨어는 `IApplicationBuilder.UseMiddleware<T>()`를 통해 파이프라인에 추가한다. 편의성을 위해 확장 메서드를 사용하는 것이 일반적이다.

```csharp
// Program.cs
using MiddlewareExample.CustomMiddleware; // 확장 메서드 사용을 위한 네임스페이스 참조

var builder = WebApplication.CreateBuilder(args);

// IMiddleware를 상속하지 않는 미들웨어는 여기에 별도 DI 등록이 필요하지 않다.
// builder.Services.AddTransient<HelloCustomMiddleware>(); // 이 줄은 필요 없다.

var app = builder.Build();

app.Use(async (HttpContext context, RequestDelegate next) => // middleware 1
{
    await context.Response.WriteAsync("hello 1\n"); // 첫 번째 미들웨어 진입 시 출력
    await next(context); // 다음 미들웨어 호출
});

// 커스텀 미들웨어(HelloCustomMiddleware)를 파이프라인에 추가한다.
app.UseHelloCustomMiddleware(); // HelloCustomMiddlewareExtensions에 정의된 확장 메서드 호출

app.Run(async (HttpContext context) => // middleware 2 (터미널 미들웨어)
{
    await context.Response.WriteAsync("hello 2\n"); // 파이프라인의 끝에서 출력
});

app.Run(); // 애플리케이션 실행
```

#### 파이프라인 설명

1. **`app.Use(...)` (Middleware 1)**: 요청이 들어오면 가장 먼저 실행된다. "hello 1"을 출력하고 `next(context)`를 호출하여 다음 미들웨어로 넘어간다.
2. **`app.UseHelloCustomMiddleware()`**: `HelloCustomMiddleware`가 실행된다.
    - 쿼리 문자열(`?firstname=John&lastname=Doe`)에서 `firstname`과 `lastname`을 확인한다.
    - 두 값이 모두 존재하면 `John Doe`와 같은 전체 이름을 JSON 형태로 응답에 기록한다. (참고: `WriteAsJsonAsync` 사용 시 `Content-Type: application/json` 헤더가 설정되며, 이 미들웨어에서 응답이 완료될 수 있어 이후 `WriteAsync` 호출이 무시될 수 있다.)
    - `_next(httpContext)`를 호출하여 다음 미들웨어로 넘어간다.
3. **`app.Run(...)` (Middleware 2)**: 파이프라인의 마지막 미들웨어로 실행된다. "hello 2"를 출력한다. `app.Run()`은 파이프라인을 종료하므로, 이 시점에서 응답이 최종적으로 클라이언트로 전달된다.
4. **`HelloCustomMiddleware`로 제어 복귀**: Middleware 2가 완료되면 `HelloCustomMiddleware`의 `_next(httpContext)` 호출 이후 로직으로 제어가 돌아온다.
5. **`Middleware 1`로 제어 복귀**: `HelloCustomMiddleware`가 완료되면 Middleware 1의 `next(context)` 호출 이후 로직으로 제어가 돌아온다.

---

### 3. 실행 시나리오 및 결과

`Program.cs`를 실행한 후 브라우저에서 다음 URL로 접근한다.

```
http://localhost:[포트번호]/?firstname=John&lastname=Doe
```

**응답 순서**:

1. Middleware 1에서 "hello 1"이 출력된다.
2. `HelloCustomMiddleware`에서 "John Doe"가 JSON 형식으로 출력된다. (이는 `WriteAsJsonAsync`가 응답을 완료하여 이후 `WriteAsync`가 무시될 가능성이 있다.)
3. `_next(httpContext)` 호출 이후의 After Logic이 실행될 것이다.

---

### 4. 정리: 두 미들웨어 방식의 차이점

| 특징               | `IMiddleware` 인터페이스 구현 방식 (`MyCustomMilddleware`)                                                                  | 규약 기반 미들웨어 (`HelloCustomMiddleware`)                                                                                                  |
| ---------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| **인터페이스 구현**     | `IMiddleware` 인터페이스를 명시적으로 구현한다.                                                                                   | 어떤 인터페이스도 구현하지 않는다.                                                                                                                   |
| **생성자**          | 일반적으로 의존성(예: 서비스)만 주입받는다. `RequestDelegate`는 받지 않는다.                                                               | `RequestDelegate next`를 **생성자 매개변수**로 주입받는다.                                                                                          |
| **`Invoke` 메서드** | `public async Task InvokeAsync(HttpContext context, RequestDelegate next)`로 정의한다. `context`와 `next`를 모두 매개변수로 받는다. | `public async Task Invoke(HttpContext context)` (또는 `InvokeAsync`)로 정의한다. `context`만 매개변수로 받는다. 다음 미들웨어는 생성자에서 주입받은 `_next` 필드를 사용한다. |
| **DI 컨테이너 등록**   | `builder.Services.AddTransient<MyCustomMilddleware>()`와 같이 명시적으로 등록해야 한다.                                          | `app.UseMiddleware<HelloCustomMiddleware>()` 호출 시 자동으로 등록된다.                                                                          |
| **장점**           | 명확한 인터페이스 기반 구조                                                                                                    | 더 간결한 코드, 인터페이스 상속 불필요. ASP.NET Core 6부터 권장되는 방식이다.                                                                                   |
