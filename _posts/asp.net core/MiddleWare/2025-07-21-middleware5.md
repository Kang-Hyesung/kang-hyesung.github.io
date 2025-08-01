---
title: "[5] 인터페이스 없는 미들웨어 생성 및 활용"
date: 2025-07-21 23:03 +0900
author: hyesung
---

ASP.NET Core에서 미들웨어(Middleware)는 HTTP 요청과 응답을 처리하는 파이프라인을 구성하는 핵심 요소다. 일반적으로 `IMiddleware` 인터페이스를 구현하여 미들웨어를 정의하지만, 특정 규약(Convention)을 따르기만 하면 인터페이스 없이도 클래스를 미들웨어로 만들 수 있다. 이 방식은 코드를 더 간결하게 만들고, 최신 ASP.NET Core에서 권장되는 접근 방식이다.

-----

### 1\. 규약 기반 미들웨어의 핵심 규칙

`IMiddleware` 인터페이스를 구현하지 않는 일반 C\# 클래스가 미들웨어로 동작하기 위해서는 다음 두 가지 핵심 규약을 반드시 따라야 한다.

1.  **생성자 (Constructor)**: 반드시 `RequestDelegate` 타입의 매개변수를 하나 가져야 한다. 이 매개변수를 통해 파이프라인의 다음 미들웨어를 주입받는다.
2.  **`Invoke` 또는 `InvokeAsync` 메서드**: HTTP 요청을 실제로 처리하는 메서드다. 반드시 `public` 접근 제한자를 가져야 하며, 첫 번째 매개변수로 `HttpContext`를 받아야 한다. 비동기 처리가 필요하면 `Task`를 반환하는 `InvokeAsync`로, 동기 처리가 필요하면 `void`를 반환하는 `Invoke`로 정의할 수 있다.

-----

### 2\. 핵심 코드 구현 (C\#)

위 규칙을 바탕으로, 쿼리 문자열에 `firstname`과 `lastname`이 있는지 확인하고 이를 조합하여 응답하는 간단한 미들웨어를 작성한다.

#### 📄 HelloCustomMiddleware.cs

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;
using System.Threading.Tasks;

namespace MiddlewareExample
{
    // 특정 인터페이스를 구현하지 않는 일반 클래스다.
    public class HelloCustomMiddleware
    {
        // 1. 다음 미들웨어를 가리키는 RequestDelegate 필드를 선언한다.
        private readonly RequestDelegate _next;

        // 2. 생성자를 통해 다음 미들웨어(RequestDelegate)를 주입받는다.
        public HelloCustomMiddleware(RequestDelegate next)
        {
            _next = next;
        }

        // 3. 'InvokeAsync' 메서드를 정의하고 HttpContext를 매개변수로 받는다.
        public async Task InvokeAsync(HttpContext httpContext)
        {
            // [요청 처리 전 로직 (Before Logic)]
            // 쿼리 문자열에 'firstname'과 'lastname' 키가 모두 있는지 확인한다.
            if (httpContext.Request.Query.ContainsKey("firstname") && httpContext.Request.Query.ContainsKey("lastname"))
            {
                var firstName = httpContext.Request.Query["firstname"];
                var lastName = httpContext.Request.Query["lastname"];

                // 두 값이 모두 있으면 전체 이름을 구성하여 JSON 형식으로 응답에 기록한다.
                await httpContext.Response.WriteAsJsonAsync(new { fullName = $"{firstName} {lastName}" });
                
                // 여기서 응답이 끝났으므로 다음 미들웨어를 호출하지 않고 파이프라인을 중단한다.
                return; 
            }

            // 다음 미들웨어 호출
            await _next(httpContext);

            // [요청 처리 후 로직 (After Logic)]
            // 다음 미들웨어의 처리가 완료된 후 실행된다.
        }
    }

    // 미들웨어를 HTTP 요청 파이프라인에 편리하게 추가하기 위한 확장 메서드다.
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

**주요 특징**:

  * **생성자 주입**: ASP.NET Core는 해당 미들웨어의 인스턴스를 생성할 때 자동으로 다음 미들웨어 체인을 `next` 매개변수로 전달한다.
  * **`InvokeAsync` 메서드**: HTTP 요청을 처리하는 핵심 로직을 담고 있으며, `HttpContext`를 통해 요청과 응답에 접근한다.
  * **`_next(httpContext)` 호출**: 이 호출을 통해 제어권을 파이프라인의 다음 미들웨어로 넘긴다. 이 호출을 기준으로 호출 전은 'Before Logic', 호출 후는 'After Logic'으로 나뉜다.
  * **DI 컨테이너 등록 불필요**: 이 방식의 미들웨어는 `Program.cs`의 서비스 컬렉션에 직접 등록할 필요가 없다. `app.UseMiddleware<T>()`가 내부적으로 미들웨어를 DI 컨테이너에 등록하고 관리하기 때문이다.

-----

### 3\. 요청 파이프라인에 미들웨어 추가하기

작성된 미들웨어는 `Program.cs`에서 확장 메서드를 통해 간결하게 등록할 수 있다.

#### 📄 Program.cs

```csharp
using MiddlewareExample; // 확장 메서드 사용을 위한 네임스페이스 참조

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Middleware 1: 간단한 인라인 미들웨어
app.Use(async (context, next) =>
{
    Console.WriteLine("Middleware 1: Before next");
    await next(context);
    Console.WriteLine("Middleware 1: After next");
});

// Middleware 2: 우리가 만든 커스텀 미들웨어
app.UseHelloCustomMiddleware();

// Middleware 3: 파이프라인의 끝을 담당하는 터미널 미들웨어
app.Run(async (context) =>
{
    Console.WriteLine("Terminal Middleware reached.");
    await context.Response.WriteAsync("Hello from Terminal Middleware!");
});

app.Run();
```

위 파이프라인의 동작은 다음과 같다.

1.  요청이 들어오면 **Middleware 1**이 실행된다.
2.  `next(context)` 호출을 통해 제어가 **`HelloCustomMiddleware`** 로 넘어간다.
3.  `HelloCustomMiddleware`는 쿼리 문자열을 검사한다.
      * **조건 충족 시 (`?firstname=...&lastname=...`)**: JSON 응답을 작성하고 `return`을 통해 파이프라인을 종료한다. **Middleware 3**은 실행되지 않는다.
      * **조건 불충족 시**: `_next(httpContext)`를 호출하여 제어를 **Middleware 3 (`app.Run`)** 으로 넘긴다.
4.  Middleware 3이 실행된 후, 제어는 다시 `HelloCustomMiddleware`의 `_next` 호출 이후로, 그다음 Middleware 1의 `next` 호출 이후로 돌아와 요청 처리가 완료된다.

-----

### 4\. Java Spring에서는? `Filter`의 역할

ASP.NET Core의 미들웨어와 거의 동일한 역할을 하는 것이 Java Spring 생태계에는 **서블릿 필터(Servlet Filter)** 가 있다. 필터는 클라이언트의 요청이 서블릿(Spring의 `DispatcherServlet`)에 도달하기 전후에 다양한 전처리 및 후처리 작업을 수행할 수 있는 강력한 컴포넌트다.

| ASP.NET Core Middleware | Java Spring Filter                                   | 설명                                                           |
| :---------------------- | :--------------------------------------------------- | :------------------------------------------------------------- |
| `HttpContext`           | `HttpServletRequest`, `HttpServletResponse`          | HTTP 요청/응답 정보를 담고 있는 객체                           |
| `RequestDelegate next`  | `FilterChain chain`                                  | 다음 필터 또는 서블릿으로 요청을 전달하는 체인 객체            |
| `await next(context)`   | `chain.doFilter(request, response)`                  | 다음 컴포넌트로 제어를 넘기는 호출                             |
| `app.Use...()`          | `@Component`, `@Order` 또는 `FilterRegistrationBean` | 파이프라인/필터 체인에 필터를 등록하는 방법                    |
| `InvokeAsync` 메서드    | `doFilter` 메서드                                    | 필터의 핵심 로직을 구현하는 메서드                             |

#### ☕️ Spring Filter 예제 코드 (Java)

위 C\# 예제와 유사하게, 특정 요청 파라미터가 있는지 확인하는 Spring Filter 예제 코드다.

```java
import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
@Order(1) // 필터의 실행 순서를 지정한다. 숫자가 낮을수록 먼저 실행된다.
public class CustomParameterCheckFilter implements Filter {

    private static final Logger logger = LoggerFactory.getLogger(CustomParameterCheckFilter.class);

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;

        // [요청 처리 전 로직 (Before Logic)]
        logger.info("CustomParameterCheckFilter: Before chain.doFilter()");

        String firstName = httpRequest.getParameter("firstname");
        String lastName = httpRequest.getParameter("lastname");

        if (firstName != null && lastName != null) {
            logger.warn("Request contains sensitive parameters. Halting chain.");
            // 여기서 응답을 직접 작성하고 체인을 중단할 수 있다.
            // response.getWriter().write("Parameters detected.");
            // return;
        }

        // 다음 필터 또는 서블릿으로 요청을 전달한다.
        chain.doFilter(request, response);

        // [요청 처리 후 로직 (After Logic)]
        logger.info("CustomParameterCheckFilter: After chain.doFilter()");
    }
}
```

이 Java 코드는 `doFilter` 메서드 내에서 `chain.doFilter()`를 호출하는 것을 기준으로 요청 전/후 로직을 구현하며, `@Component` 어노테이션을 통해 스프링이 이 클래스를 빈(Bean)으로 자동 등록하게 만든다.

-----

### 5\. `IMiddleware` 방식 vs 규약 기반 방식 비교

마지막으로 ASP.NET Core 내에서 두 가지 미들웨어 정의 방식을 비교한다.

| 특징             | `IMiddleware` 인터페이스 구현 방식                                 | 규약 기반 미들웨어                                                  |
| :--------------- | :----------------------------------------------------------------- | :------------------------------------------------------------------ |
| **인터페이스 구현** | `IMiddleware` 인터페이스를 명시적으로 구현해야 한다.                 | 어떤 인터페이스도 구현할 필요가 없다.                                 |
| **생성자** | 일반적으로 의존성(서비스)만 주입받는다. `RequestDelegate`는 받지 않는다. | `RequestDelegate next`를 **생성자 매개변수**로 반드시 주입받아야 한다.  |
| **`Invoke` 메서드** | `InvokeAsync(HttpContext context, RequestDelegate next)`로 정의한다. | `InvokeAsync(HttpContext context)`로 정의한다. `next`는 필드 멤버를 사용한다. |
| **DI 컨테이너 등록** | `builder.Services.AddTransient<T>()`와 같이 명시적으로 등록해야 한다. | `app.UseMiddleware<T>()` 호출 시 자동으로 처리된다.                   |
| **장점** | 명확한 인터페이스 계약(Contract) 기반의 구조로 예측 가능하다.        | 코드가 더 간결하며, ASP.NET Core 6부터 권장되는 방식이다.             |

### 결론

ASP.NET Core의 규약 기반 미들웨어는 `IMiddleware` 인터페이스의 제약에서 벗어나 더 유연하고 간결하게 코드를 작성할 수 있는 강력한 방법이다. 이는 Java Spring의 `Filter`와 개념적으로 매우 유사하다. 어떤 방식을 선택하든, 미들웨어(필터)는 로깅, 인증, 예외 처리 등 애플리케이션의 공통 관심사(Cross-cutting Concerns)를 처리하는 데 필수적인 도구라는 점을 기억하는 것이 중요하다.