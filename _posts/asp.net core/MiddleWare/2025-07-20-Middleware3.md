---
title: "[3] Custom Middleware 등록"
date: 2025-07-20 23:13 +0900
author: hyesung
description: 설명
---
## 내용 정리
### 1. DI 컨테이너에 커스텀 미들웨어 등록

```csharp
var builder = WebApplication.CreateBuilder(args);
// Transient: 매 요청마다 MyCustomMilddleware 인스턴스 새로 생성
builder.Services.AddTransient<MyCustomMilddleware>();
```

### 2. 클래스 기반 미들웨어 삽입 (`UseMiddleware<T>()`)

```csharp
var app = builder.Build();
// 파이프라인의 미들웨어로 삽입
app.UseMiddleware<MyCustomMilddleware>();
```

- `UseMiddleware<T>()`는 `IMiddleware` 구현 클래스를 삽입
- 순서에 따라 실행 순서 결정

### 3. Terminating Middleware (`app.Run`)

```csharp
// 인수 1개(HttpContext)만 받으며, 무조건 파이프라인 종료
app.Run(async context =>
{
    await context.Response.WriteAsync("hello 2");
});

// 서버 실행
app.Run();
```

- `Run` 뒤에 오는 미들웨어는 실행되지 않음

### 4. MyCustomMilddleware.cs (커스텀 미들웨어 구현)

```csharp
using Microsoft.AspNetCore.Http;
using System.Threading.Tasks;

namespace MiddlewareExample.CustomMiddleware
{
    // 여기서 IMiddleware 구현해야 UseMiddleware 메소드에서 사용 가능
    public class MyCustomMilddleware : IMiddleware
    {
        // IMiddleware 구현한 클래스는 InvokeAsync 도 반드시 구현해야 함
        public async Task InvokeAsync(HttpContext context, RequestDelegate next)
        {
            // Before 로직: 요청이 파이프라인에 진입할 때 실행
            await context.Response.WriteAsync("Custom Middleware Executed!\n");

            // 다음 미들웨어로 제어 전달
            await next(context);

            // After 로직: 후속 미들웨어 처리 완료 후 실행
            await context.Response.WriteAsync("Custom Middleware Completed!\n");
        }
    }
}
```

- `IMiddleware` 구현 → `InvokeAsync(HttpContext, RequestDelegate)` 필수
- `context`로 HTTP 요청·응답 접근
- `next(context)` 호출 전/후에 원하는 로직 삽입 가능

### 5. 미들웨어 순서 및 흐름 예시

```
클라이언트 → [Use 1] → [MyCustomMilddleware (before)] → [Run("hello 2")] → 즉시 응답
                                     ↑
                                  (after)
```

1. **Use 1** (예: 로깅)
2. **MyCustomMilddleware** before 로직
3. **Run** (terminating) → `"hello 2"`
4. **MyCustomMilddleware** after 로직 (Response 스트림에 추가)
5. 최종 응답 클라이언트 반환

---
## 요약

- DI에 `AddTransient<T>()`로 등록 → `UseMiddleware<T>()`로 삽입
- `app.Run(...)`은 무조건 terminating → 이후 미들웨어 실행 중단
- 복잡한 로직은 `IMiddleware` 구현 클래스로 분리
- 순서와 호출 여부(`next`)가 파이프라인 제어의 핵심



네, 알겠습니다. 요청하신 대로 미들웨어의 실행 흐름을 더 직관적으로 이해할 수 있도록 그림 형태의 설명을 보강하여 블로그 글을 다시 작성해 드리겠습니다.

-----

# [C\#] 커스텀 미들웨어(Custom Middleware) 등록과 실행 흐름 완벽 분석

## 도입

모던 웹 프레임워크에서 미들웨어(Middleware)는 요청과 응답을 처리하는 파이프라인을 구성하는 핵심적인 소프트웨어 컴포넌트다. ASP.NET Core는 이러한 미들웨어를 통해 인증, 로깅, 예외 처리 등 다양한 공통 기능을 모듈화하여 관리한다.

이번 포스트에서는 ASP.NET Core에서 커스텀 미들웨어를 직접 구현하고, DI 컨테이너에 등록하여 파이프라인에 삽입하는 방법을 알아본다. 또한, 미들웨어의 실행 순서와 제어 흐름을 심도 있게 분석하여 동작 원리를 명확히 이해하는 것을 목표로 한다.

-----

## 1\. 커스텀 미들웨어 클래스 구현하기

복잡한 로직을 처리하거나 재사용성을 높이기 위해, 미들웨어를 별도의 클래스로 분리하는 것이 좋다. ASP.NET Core에서 클래스 기반 미들웨어를 만들려면 **`IMiddleware`** 인터페이스를 구현해야 한다.

  - **`IMiddleware`**: `UseMiddleware<T>()` 확장 메서드를 통해 미들웨어를 파이프라인에 등록하기 위해 필요한 인터페이스다.
  - **`InvokeAsync`**: `IMiddleware`를 구현하면 반드시 포함해야 하는 메서드다. HTTP 요청이 들어올 때마다 호출되며, 실제 미들웨어 로직이 이 안에 담긴다.

<!-- end list -->

```csharp
// MyCustomMiddleware.cs
using Microsoft.AspNetCore.Http;
using System.Threading.Tasks;

namespace MiddlewareExample.CustomMiddleware
{
    /// <summary>
    /// IMiddleware를 구현하여 클래스 기반 미들웨어를 정의한다.
    /// </summary>
    public class MyCustomMiddleware : IMiddleware
    {
        /// <summary>
        /// 미들웨어의 핵심 로직을 포함하는 메서드.
        /// </summary>
        /// <param name="context">HTTP 요청/응답에 대한 정보를 담고 있는 객체</param>
        /// <param name="next">파이프라인의 다음 미들웨어를 호출하는 대리자(Delegate)</param>
        public async Task InvokeAsync(HttpContext context, RequestDelegate next)
        {
            // 1. 'next' 호출 전: 요청(Request) 단계에서 실행되는 로직
            await context.Response.WriteAsync("Custom Middleware Executed!\n");

            // 2. 다음 미들웨어로 제어 전달
            // 만약 이 코드를 호출하지 않으면, 파이프라인은 여기서 멈춘다(Short-circuit).
            await next(context);

            // 3. 'next' 호출 후: 응답(Response) 단계에서 실행되는 로직
            await context.Response.WriteAsync("Custom Middleware Completed!\n");
        }
    }
}
```

`InvokeAsync` 메서드 내부의 `await next(context)` 호출을 기준으로, 코드 실행 시점이 나뉜다.

  - **호출 전 로직**: 요청이 파이프라인을 따라 안쪽으로 들어갈 때 실행된다.
  - **호출 후 로직**: 파이프라인의 끝에서 응답이 생성되어 바깥쪽으로 나올 때 실행된다.

-----

## 2\. DI 컨테이너 등록 및 파이프라인 삽입

작성한 미들웨어 클래스는 의존성 주입(Dependency Injection, DI) 컨테이너에 서비스로 등록해야 하며, 그 후에 요청 처리 파이프라인에 추가할 수 있다.

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// 1. DI 컨테이너에 커스텀 미들웨어 등록
// AddTransient: HTTP 요청이 발생할 때마다 MyCustomMiddleware의 새 인스턴스를 생성한다.
builder.Services.AddTransient<MyCustomMiddleware>();

var app = builder.Build();

// 2. 요청 처리 파이프라인에 미들웨어 삽입
// UseMiddleware<T>를 사용해 DI 컨테이너에 등록된 미들웨어를 파이프라인에 추가한다.
// 등록된 순서가 곧 실행 순서다.
app.UseMiddleware<MyCustomMiddleware>();

// 3. 종단(Terminating) 미들웨어 등록
// Run 메서드는 RequestDelegate를 인수로 받아 파이프라인을 종료한다.
// 이 미들웨어는 'next'를 호출하지 않으므로, 이 뒤에 오는 미들웨어는 실행되지 않는다.
app.Run(async context =>
{
    await context.Response.WriteAsync("Terminating Middleware Executed!\n");
});

// 서버 실행
app.Run();
```

-----

## 3\. 미들웨어 실행 흐름 분석

위 코드의 실행 흐름을 요청과 응답 관점에서 도식화하면 다음과 같다. 미들웨어 파이프라인은 **요청이 들어가는 흐름**과 **응답이 되돌아 나오는 흐름**으로 구성된 양방향 통로와 같다.

```
[클라이언트] <=> [웹 서버]

  [요청 Request]
       |
       v
[MyCustomMiddleware] --- 1. "before" 로직 실행: "Custom Middleware Executed!"
       |
       v
[Run Middleware]   --- 2. 종단 로직 실행: "Terminating Middleware Executed!"
       |                 (여기서 응답이 생성되어 되돌아가기 시작)
       ^
       |
[MyCustomMiddleware] --- 3. "after" 로직 실행: "Custom Middleware Completed!"
       ^
       |
  [응답 Response]
```

1.  **요청 진입**: 클라이언트의 요청이 `MyCustomMiddleware`에 도달하여 `next()` 이전의 **`before`** 로직이 실행된다.
2.  **종단 처리**: 제어가 다음 미들웨어인 `app.Run()`으로 넘어가 응답을 생성한다.
3.  **응답 반환**: 생성된 응답이 파이프라인을 거슬러 올라오면서, `MyCustomMiddleware`의 `next()` 이후의 **`after`** 로직이 실행된다.
4.  **최종 전송**: 모든 `after` 로직이 완료된 최종 응답이 클라이언트로 전송된다.

**최종 클라이언트 출력 결과:**

```
Custom Middleware Executed!
Terminating Middleware Executed!
Custom Middleware Completed!
```

-----

## Java Spring에서는?

ASP.NET Core의 미들웨어와 유사한 개념으로 Java Spring에는 \*\*서블릿 필터(Servlet Filter)\*\*와 \*\*스프링 인터셉터(Spring Interceptor)\*\*가 존재한다.

### 기술 스택 매핑

| ASP.NET Core | Java Spring | 범위 | 주요 목적 |
| :--- | :--- | :--- | :--- |
| **Middleware** | **`Filter`** | Servlet Container | 요청/응답을 서블릿에 도달하기 전/후에 가로챔 (인코딩, 보안) |
| **Middleware** | **`HandlerInterceptor`** | Spring MVC | 디스패처 서블릿(Dispatcher Servlet)과 컨트롤러 사이에서 요청을 가로챔 (인증, 로깅) |

### 1\. 서블릿 필터 (`Filter`)

Spring Boot 환경에서 `Filter`는 ASP.NET Core의 미들웨어와 가장 유사한 역할을 한다. `jakarta.servlet.Filter` 인터페이스를 구현하며, `doFilter` 메서드는 미들웨어의 `InvokeAsync`와 거의 동일한 구조를 가진다.

```java
// CustomLoggingFilter.java
import org.springframework.stereotype.Component;
import jakarta.servlet.*;
import java.io.IOException;

@Component
public class CustomLoggingFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        // 1. 'chain.doFilter' 호출 전: 요청(Request) 단계 로직
        System.out.println("Before Filter Logic");

        // 2. 다음 필터 또는 서블릿으로 제어 전달 (ASP.NET의 next(context)와 유사)
        chain.doFilter(request, response);

        // 3. 'chain.doFilter' 호출 후: 응답(Response) 단계 로직
        System.out.println("After Filter Logic");
    }
}
```

  - **`doFilter(request, response, chain)`**: 미들웨어의 `InvokeAsync`에 해당한다.
  - **`chain.doFilter(...)`**: 미들웨어의 `await next(context)`와 동일한 역할을 수행하며, 다음 필터로 체인을 이어간다.
  - **`@Component`** 어노테이션을 통해 자동으로 스프링 빈(Bean)으로 등록되어 필터 체인에 추가된다.

### 2\. 스프링 인터셉터 (`HandlerInterceptor`)

인터셉터는 Spring MVC 프레임워크 내에서 더 정밀한 제어를 제공한다. 컨트롤러의 핸들러 메서드 실행 전, 후, 그리고 뷰(View) 렌더링 후 등 더 세분화된 시점에서 로직을 실행할 수 있다.

```java
// LoggerInterceptor.java
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@Component
public class LoggerInterceptor implements HandlerInterceptor {

    // 컨트롤러 메서드 실행 직전에 호출
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle: Controller 실행 전");
        return true; // true를 반환해야 다음 단계 진행
    }

    // 컨트롤러 메서드 실행 직후, 뷰 렌더링 전에 호출
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle: Controller 실행 후, View 렌더링 전");
    }

    // 뷰 렌더링까지 완료된 후 호출
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion: 요청 처리 완료 후");
    }
}
```

  - **`preHandle`**: `next()` 이전 로직과 유사하다. `false`를 반환하면 요청 처리를 중단시킬 수 있다.
  - **`postHandle`**, **`afterCompletion`**: `next()` 이후 로직과 유사하지만, 실행 시점이 더 구체적으로 나뉜다.

인터셉터를 등록하려면 `WebMvcConfigurer`를 구현한 설정 클래스가 필요하다.

```java
// WebConfig.java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private LoggerInterceptor loggerInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loggerInterceptor);
    }
}
```

-----

## 결론

ASP.NET Core의 미들웨어 파이프라인은 요청과 응답의 흐름을 명확하게 제어할 수 있는 강력한 아키텍처다. `next` 대리자를 기준으로 요청과 응답 시점의 로직을 분리할 수 있다는 점이 핵심이다.

Java Spring에서는 서블릿 필터가 이와 가장 유사한 개념이며, Spring MVC 레벨에서는 핸들러 인터셉터를 통해 더 세분화된 제어가 가능하다. 두 프레임워크 모두 웹 애플리케이션의 공통 관심사(Cross-cutting concerns)를 효과적으로 처리하기 위한 파이프라인/체인 메커니즘을 제공한다는 점에서 동일한 철학을 공유한다. 프레임워크의 차이를 이해하고 각 메커니즘의 장단점을 파악하면 더 견고하고 유연한 애플리케이션을 설계할 수 있다.