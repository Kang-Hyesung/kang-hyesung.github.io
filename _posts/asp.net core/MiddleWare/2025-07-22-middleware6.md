---
title: "[6] UseWhen 을 이용한 조건부 실행"
date: 2025-07-22 17:04 +0900
author: hyesung
---

## [ASP.NET Core] 조건에 따라 미들웨어 실행하기: Use vs UseWhen

모든 웹 애플리케이션은 들어오는 요청(Request)을 처리하고 응답(Response)을 보내는 일련의 과정을 거친다. ASP.NET Core에서는 이 과정을 **미들웨어 파이프라인(Middleware Pipeline)** 을 통해 관리하며, 개발자는 이 파이프라인에 필요한 기능들을 미들웨어 형태로 추가할 수 있다.

일반적으로 `Use` 확장 메서드를 사용하여 모든 요청에 대해 실행될 미들웨어를 파이프라인에 등록한다. 하지만 특정 조건에서만 미들웨어를 실행하고 싶을 때도 있다. 예를 들어, 특정 URL 경로에만 로깅을 적용하거나, 요청 헤더에 인증 토큰이 있을 때만 사용자 인증을 처리하는 경우다.

이럴 때 사용하는 것이 바로 `UseWhen` 메서드다. 이번 글에서는 `UseWhen`이 무엇이며 `Use`와 어떻게 다른지, 그리고 어떤 상황에서 유용하게 사용할 수 있는지 알아본다.

### Use vs. UseWhen: 핵심 차이점

두 메서드 모두 미들웨어를 요청 파이프라인에 추가하지만, 실행되는 방식에 근본적인 차이가 있다.

  - **`Use`**: 파이프라인에 등록된 순서에 따라 **모든 요청**에 대해 미들웨어를 실행한다. 가장 일반적인 미들웨어 등록 방식이다.
  - **`UseWhen`**: 특정 조건(`Predicate`)을 만족하는 요청에 대해서만 미들웨어의 **분기(Branch)** 를 실행한다. 조건이 만족되지 않으면 해당 분기는 건너뛴다.

-----

### `UseWhen`의 작동 방식 🔎

`UseWhen`은 조건부로 파이프라인을 분기하여 특정 로직을 실행한 뒤, 다시 원래의 주 파이프라인으로 흐름을 되돌리는 방식으로 동작한다.

1.  **초기 미들웨어 실행**: 요청이 파이프라인에 진입하면 `UseWhen` 이전의 미들웨어들이 순서대로 실행된다.
2.  **`UseWhen` 조건 확인**: `UseWhen`에 정의된 조건(예: 요청 헤더 값, 쿼리 문자열, 경로 등)을 확인한다. 이 조건은 `HttpContext`를 인자로 받아 `bool` 값을 반환하는 람다식으로 정의된다.
3.  **조건부 미들웨어 분기 실행**:
      * **조건이 참(true)일 경우**: `UseWhen`에 정의된 미들웨어 분기가 실행된다. 이 분기 파이프라인이 모두 실행된 후, 요청은 다시 **주 체인(Main Chain)** 으로 돌아가 다음 미들웨어를 계속 실행한다.
      * **조건이 거짓(false)일 경우**: 미들웨어 분기를 건너뛰고, 요청은 즉시 주 체인의 다음 미들웨어로 넘어간다.

여기서 \*\*주 체인(Main Chain)\*\*이란, 조건과 관계없이 모든 요청이 공통으로 거치는 핵심 미들웨어 파이프라인을 의미한다.

#### 예제 코드로 살펴보기

`Program.cs` 파일에 다음과 같이 미들웨어를 구성하여 `UseWhen`의 동작을 확인할 수 있다.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// 'username' 쿼리 파라미터가 있을 때만 실행되는 미들웨어 분기
app.UseWhen(
    // 1. 조건: 요청 쿼리에 "username" 키가 있는지 확인한다.
    context => context.Request.Query.ContainsKey("username"),

    // 2. 미들웨어 분기: 조건이 참일 때 실행할 파이프라인을 구성한다.
    appBranch => {
        appBranch.Use(async (context, next) =>
        {
            await context.Response.WriteAsync("Hello from Middleware branch. ");
            await next(); // 분기 내 다음 미들웨어 또는 주 체인으로 제어를 넘긴다.
        });
    });

// 3. 주 체인: UseWhen 조건과 관계없이 항상 실행된다.
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello from middleware at main chain.");
});

app.Run();
```

#### 실행 결과 분석

  - **`http://localhost:5000`으로 요청 시:**

      - `username` 쿼리 파라미터가 없으므로 `UseWhen`의 조건은 `false`다.
      - 미들웨어 분기는 실행되지 않고 `app.Run` 미들웨어만 실행된다.
      - **응답**: `Hello from middleware at main chain.`

  - **`http://localhost:5000/?username=gemini`으로 요청 시:**

      - `username` 쿼리 파라미터가 있으므로 `UseWhen`의 조건은 `true`다.
      - `UseWhen`의 미들웨어 분기가 실행되어 첫 번째 응답을 작성한다.
      - 분기 실행 후, 주 체인으로 돌아와 `app.Run` 미들웨어가 실행된다.
      - **응답**: `Hello from Middleware branch. Hello from middleware at main chain.`

-----

### 언제 `UseWhen`을 사용해야 할까?

`UseWhen`은 다음과 같이 요청의 특정 속성에 따라 다른 파이프라인 로직을 적용해야 할 때 매우 유용하다.

  - **인증 분기 처리**: 요청 헤더에 `Authorization` 토큰이 존재할 때만 인증 및 권한 부여 미들웨어를 실행할 수 있다.
  - **특정 경로 로깅**: `/api` 와 같이 특정 경로로 시작하는 요청에 대해서만 상세한 요청/응답 로깅 미들웨어를 적용할 수 있다.
  - **A/B 테스팅**: 특정 쿠키나 헤더 값을 기준으로 사용자 그룹을 나누어 서로 다른 미들웨어 파이프라인을 타게 하여 새로운 기능을 테스트할 수 있다.

-----

### Java Spring에서는? 🤔

Java Spring 환경, 특히 Spring Boot에서는 ASP.NET Core의 `UseWhen`과 같이 파이프라인 자체를 분기하는 명시적인 메서드는 없다. 대신, **필터(Filter)** 또는 **인터셉터(HandlerInterceptor)** 내부에서 조건부 로직을 구현하는 방식으로 동일한 목표를 달성한다.

Spring의 `Filter`는 `DispatcherServlet`에 도달하기 전 모든 요청을 가로채는 역할을 한다. 개발자는 커스텀 필터를 만들고, 그 필터의 `doFilter` 메서드 안에서 `if` 문을 사용하여 원하는 조건을 검사할 수 있다.

#### Spring Boot Filter 예제

ASP.NET Core 예제와 유사하게, `username` 쿼리 파라미터가 있을 때만 특정 로직을 실행하는 `Filter`를 만들어보자.

**1. 커스텀 필터 작성 (`ConditionalLogicFilter.java`)**

```java
import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class ConditionalLogicFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;

        // C# UseWhen의 조건과 동일한 로직
        if (httpRequest.getParameter("username") != null) {
            // 조건이 참일 때 실행할 로직
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            httpResponse.getWriter().write("Hello from Filter's conditional branch. ");
        }

        // 주 체인(Main Chain)으로 요청을 계속 전달한다.
        chain.doFilter(request, response);
    }
}
```

**2. 컨트롤러 작성 (`MainController.java`)**

Spring에서 `app.Run`과 비슷한 역할은 컨트롤러의 핸들러 메서드가 수행한다.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MainController {

    @GetMapping("/")
    public String handleMainRequest() {
        // 필터 실행 후, 주 체인에 해당하는 로직
        return "Hello from Controller at main chain.";
    }
}
```

#### 동작 방식 비교

  - **ASP.NET Core `UseWhen`**: 프레임워크 수준에서 파이프라인의 **구조 자체를 분기**한다.
  - **Java Spring `Filter`**: 단일 필터 **내부에서 `if` 문을 통해 로직을 분기**한다. 요청은 모든 필터 체인을 통과하지만, 필터 안의 코드가 조건에 따라 실행될 뿐이다.

결과적으로 두 프레임워크 모두 '조건부 로직 실행'이라는 동일한 목표를 달성하지만, 그 접근 방식과 철학에 차이가 있음을 알 수 있다.

### 결론

`UseWhen`은 ASP.NET Core에서 동적으로 요청 파이프라인을 구성할 수 있는 강력하고 유연한 도구다. 모든 요청에 동일한 미들웨어를 적용하는 대신, 특정 조건에 따라 필요한 로직만 선택적으로 실행함으로써 더 깔끔하고 효율적인 코드를 작성할 수 있다.