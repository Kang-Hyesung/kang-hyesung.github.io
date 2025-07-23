---
title: "[7] Middleware 단원 총정리"
date: 2025-07-22 18:06 +0900
author: hyesung
---
## ASP.NET Core 미들웨어: 요청 처리 파이프라인 이해하기

ASP.NET Core에서 **미들웨어(Middleware)**는 모든 HTTP 요청과 응답이 흐르는 파이프라인을 구성하는 일련의 구성 요소다. 각 미들웨어 구성 요소는 다음을 수행할 수 있다.
- 들어오는 요청을 **검사**한다.
- 요청 또는 응답을 **수정**한다(필요한 경우).
- 파이프라인의 다음 미들웨어를 **호출**하거나, 프로세스를 **단락(short-circuit)**시키고 직접 응답을 생성한다.
    
이러한 파이프라인은 애플리케이션의 로직을 모듈화하고 인증, 로깅, 오류 처리, 라우팅과 같은 기능을 깔끔하고 유지 관리하기 쉬운 방식으로 추가할 수 있도록 해준다.

---

### 미들웨어 체인(요청 파이프라인)

요청 파이프라인을 일련의 연결된 파이프라고 상상해 봐라. 각 미들웨어는 이 파이프라인의 **밸브**와 같아서 정보의 흐름을 제어하고 다양한 단계에서 특정 작업을 적용할 수 있도록 한다. 미들웨어를 **등록하는 순서가 중요**하며, 이들은 순차적으로 실행된다.

---

### app.Use 대 app.Run

이 두 메서드는 파이프라인에 미들웨어를 추가하는 데 필수적이지만, 중요한 차이점이 있다.

- **`app.Use(async (context, next) => { ... })`**
    
    - **비-종료(Non-Terminal) 미들웨어**: 이 유형의 미들웨어는 일반적으로 어떤 작업을 수행한 다음 **`next`** 델리게이트를 호출하여 파이프라인의 다음 미들웨어로 제어를 전달한다.
    - **요청/응답 수정 가능**: 요청 또는 응답을 다음으로 전달하기 전에 변경할 수 있다.
    - **예시**: 인증, 로깅, 사용자 정의 헤더 등.
        
- **`app.Run(async (context) => { ... })`**
    
    - **종료(Terminal) 미들웨어**: 이 미들웨어는 **`next`**를 호출하지 않는다. 파이프라인을 **종료**하고 직접 응답을 생성한다.
    - **최종 응답에 자주 사용**: 추가 처리가 필요 없는 요청(예: 간단한 메시지 반환)을 처리하는 데 일반적으로 사용된다.
    - **요청 수정 불가**: 파이프라인의 끝이기 때문에 요청을 다음으로 전달하기 전에 수정할 수 없다.
        

#### 코드 1: 여러 `app.Run` 호출의 결과
```csharp
app.Run(async (HttpContext context) => {
    await context.Response.WriteAsync("Hello");
});

app.Run(async (HttpContext context) => {
    await context.Response.WriteAsync("Hello again");
});

app.Run();
```

이 코드에서는 **첫 번째 `app.Run` 미들웨어만 실행**된다. "Hello"를 응답에 작성함으로써 파이프라인을 종료하고, 다음 `app.Run` (이는 "Hello again"을 작성할 것임)은 실행될 기회를 얻지 못한다.

**설명**: **`app.Run`**은 미들웨어 체인을 **단락**시키기 때문에, 여러 개의 **`app.Run`**을 연속해서 사용하면 첫 번째 **`app.Run`**만 동작하고 그 뒤의 **`app.Run`**은 무시된다.

#### 코드 2: `app.Use`와 `app.Run`을 이용한 미들웨어 연결
```csharp
// 미들웨어 1
app.Use(async (context, next) => {
    await context.Response.WriteAsync("Hello ");
    await next(context);
});

// 미들웨어 2
app.Use(async (context, next) => {
    await context.Response.WriteAsync("Hello again ");
    await next(context);
});

// 미들웨어 3
app.Run(async (HttpContext context) => {
    await context.Response.WriteAsync("Hello again");
});
```

이 코드는 미들웨어를 올바르게 연결하는 방법을 보여준다.
1. 첫 번째 **`app.Use`**는 "Hello "를 응답에 쓰고 **`next`**를 호출하여 다음 미들웨어로 제어를 전달한다.
2. 두 번째 **`app.Use`**는 "Hello again "을 쓰고 역시 **`next`**를 호출한다.
3. 마지막 **`app.Run`** (종료 미들웨어)은 "Hello again"을 쓰고 파이프라인을 종료한다.

**결과**: "Hello Hello again Hello again"이 출력된다. 이는 각 **`app.Use`** 미들웨어가 먼저 응답을 작성한 다음 **`next`**를 통해 다음 미들웨어로 진행하고, 최종적으로 **`app.Run`**이 응답을 완료했기 때문이다.

#### 기억해야 할 핵심 사항

- **미들웨어 순서가 중요**: 미들웨어를 등록하는 순서가 중요하며, 이들은 순차적으로 실행된다.
- **비-종료 작업에는 `app.Use` 사용**: 인증, 로깅, 헤더/바디 수정과 같은 작업에 사용한다.
- **파이프라인 종료에는 `app.Run` 사용**: 최종 응답을 생성할 때 사용한다.
- **단락(Short-Circuiting)**: 미들웨어는 **`next`**를 호출하지 않고 일찍 응답을 반환하여 파이프라인을 단락시킬 수 있다.

---

### ASP.NET Core의 사용자 정의 미들웨어

ASP.NET Core는 수많은 내장 미들웨어 구성 요소를 제공하지만, 때로는 애플리케이션의 특정 요구 사항에 맞춰 자체 미들웨어를 생성해야 할 수도 있다. 사용자 정의 미들웨어를 사용하면 다음을 수행할 수 있다.

- **로직 캡슐화**: 관련 작업(예: 로깅, 보안 검사, 사용자 정의 헤더)을 재사용 가능한 구성 요소로 묶는다.
- **동작 사용자 정의**: 애플리케이션의 요구 사항에 정확히 일치하도록 요청/응답 파이프라인을 조정한다.
- **코드 구성 개선**: 미들웨어 코드를 깔끔하고 유지 관리하기 쉽게 유지한다.

#### 사용자 정의 미들웨어 클래스의 구성

- **`IMiddleware` 구현**: 이 인터페이스는 단일 메서드인 **`InvokeAsync(HttpContext context, RequestDelegate next)`**를 요구한다. 이것이 미들웨어 로직의 핵심이다.
- **`InvokeAsync` 또는 `Invoke` 메서드**:
    - **`context`**: 현재 요청을 나타내는 **`HttpContext`** 객체다. 요청 및 응답 객체에 접근할 수 있다.
    - **`next`**: **`RequestDelegate`**는 파이프라인의 다음 미들웨어를 호출할 수 있도록 한다.

#### 코드 예시 설명
```csharp
// MyCustomMiddleware.cs
namespace MiddlewareExample.CustomMiddleware
{
    public class MyCustomMiddleware : IMiddleware  // IMiddleware 인터페이스 구현
    {
        public async Task InvokeAsync(HttpContext context, RequestDelegate next)
        {
            await context.Response.WriteAsync("My Custom Middleware - Starts\n"); // 1. 시작 메시지 작성
            await next(context);                               // 2. 다음 미들웨어 호출 (중요!)
            await context.Response.WriteAsync("My Custom Middleware - Ends\n");   // 3. 종료 메시지 작성
        }
    }

    // 쉬운 등록을 위한 확장 메서드
    public static class CustomMiddlewareExtension
    {
        public static IApplicationBuilder UseMyCustomMiddleware(this IApplicationBuilder app)
        {
            return app.UseMiddleware<MyCustomMiddleware>(); // MyCustomMiddleware를 파이프라인에 추가
        }
    }
}
```

- **`MyCustomMiddleware` 클래스**: **`IMiddleware`**를 구현한다. **`InvokeAsync`** 메서드는 다음을 수행한다.
    - "My Custom Middleware - Starts"를 응답에 작성한다.
    - **`next(context)`**를 호출하여 파이프라인의 다음 미들웨어를 호출한다. **이 부분이 중요한다.** 이 호출이 없으면 파이프라인이 여기서 멈춘다.
    - 다음 미들웨어가 완료된 후 "My Custom Middleware - Ends"를 응답에 작성한다. 이 코드는 **다음 미들웨어의 실행이 끝난 후** 실행된다.
- **`CustomMiddlewareExtension` 클래스**: **`UseMyCustomMiddleware`**라는 편리한 확장 메서드를 제공하여 **`Startup.Configure`** 메서드에서 미들웨어를 쉽게 등록할 수 있도록 한다.

```csharp
// Program.cs (또는 Startup.cs)
using MiddlewareExample.CustomMiddleware;

// ...

builder.Services.AddTransient<MyCustomMiddleware>(); // 서비스로 등록 (Transient는 요청마다 새로운 인스턴스 생성)

app.Use(async (HttpContext context, RequestDelegate next) => {
    await context.Response.WriteAsync("From Midleware 1\n");
    await next(context);
});

app.UseMyCustomMiddleware(); // 확장 메서드를 사용하여 사용자 정의 미들웨어 추가

app.Run(async (HttpContext context) => {
    await context.Response.WriteAsync("From Middleware 3\n");
});
```

#### 작동 방식

- **등록**: **`MyCustomMiddleware`**를 **`Transient`** 서비스로 등록하여 ASP.NET Core가 필요할 때마다 인스턴스를 생성할 수 있도록 한다. **`Transient`**는 요청마다 새로운 인스턴스가 생성됨을 의미한다.
- **파이프라인 통합**: **`app.UseMyCustomMiddleware()`** 확장 메서드는 사용자 정의 미들웨어를 파이프라인에 원활하게 추가한다.
- **실행 순서**: 미들웨어 구성 요소는 파이프라인에 추가된 순서대로 실행된다. 이 경우 순서는 **미들웨어 1, `MyCustomMiddleware`, 그리고 미들웨어 3**이 된다.

#### 출력

애플리케이션을 실행하면 브라우저에 다음 출력이 나타난다.
```
From Midleware 1
My Custom Middleware - Starts
From Middleware 3
My Custom Middleware - Ends
```

이는 미들웨어 체인을 통한 실행 흐름을 명확하게 보여준다. **`My Custom Middleware - Ends`**가 **`From Middleware 3`** 뒤에 나오는 이유는 **`MyCustomMiddleware`**의 **`next(context)`** 호출 이후 **`From Middleware 3`**가 실행되고, **`From Middleware 3`**의 실행이 완료된 후에야 **`MyCustomMiddleware`**의 **`await next(context);`** 이후 코드가 실행되기 때문이다.

---

### 사용자 정의 Conventional 미들웨어

ASP.NET Core 미들웨어에는 두 가지 유형이 있다: 컨벤션 기반(conventional)과 팩토리 기반(factory-based). 예시에서 보여진 컨벤션 기반 미들웨어는 HTTP 요청 및 응답을 처리하기 위한 사용자 정의 로직을 캡슐화하는 간단하면서도 강력한 방법이다.

#### 주요 특징

- **클래스 기반**: 컨벤션 기반 미들웨어는 클래스로 구현된다.
- **생성자 주입**: 필요한 의존성(dependency)을 생성자를 통해 받는다.
- **`Invoke` 메서드**: 각 요청을 처리하는 로직을 포함하는 미들웨어의 핵심이다.
- **`RequestDelegate`**: **`Invoke`** 메서드는 **`RequestDelegate`** 매개변수(**`_next`**로 명명됨)를 받는다. 이 델리게이트는 파이프라인의 다음 미들웨어를 나타낸다.
- **유연성**: **`Invoke`** 메서드 내에서 요청 및 응답 객체에 대한 완전한 제어권을 가진다.
    
#### 코드 분석: `HelloCustomMiddleware`

```csharp
// HelloCustomMiddleware.cs
public class HelloCustomMiddleware
{
    private readonly RequestDelegate _next; // 다음 미들웨어를 호출하기 위한 델리게이트

    public HelloCustomMiddleware(RequestDelegate next) // 생성자를 통해 next 주입
    {
        _next = next;
    }

    public async Task Invoke(HttpContext httpContext) // 실제 로직이 담긴 Invoke 메서드
    {
        if (httpContext.Request.Query.ContainsKey("firstname") &&
            httpContext.Request.Query.ContainsKey("lastname"))
        {
            string fullName = httpContext.Request.Query["firstname"] + " " + httpContext.Request.Query["lastname"];
            await httpContext.Response.WriteAsync(fullName); // 쿼리 파라미터가 있으면 이름 출력
        }
        await _next(httpContext); // 다음 미들웨어 호출
        //By design, any code after this line, such as the comment "//after logic", would not execute for requests containing both "firstname" and "lastname", as the await _next(httpContext); line immediately transfers control to the next middleware in the pipeline.
    }
}

// 쉬운 등록을 위한 확장 메서드
public static class HelloCustomModdleExtensions
{
    public static IApplicationBuilder UseHelloCustomMiddleware(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<HelloCustomMiddleware>(); // HelloCustomMiddleware 등록
    }
}
```

각 부분을 분석해 보자:

- **생성자**: **`RequestDelegate`**를 받아 나중에 파이프라인의 다음 미들웨어를 호출하는 데 사용하도록 저장한다.
- **`Invoke` 메서드**:
    - 쿼리 문자열에 "firstname"과 "lastname" 매개변수가 모두 포함되어 있는지 확인한다.
    - 만약 그렇다면, 값을 **`fullName`** 문자열로 결합하고 이를 응답에 작성한다.
    - **결정적으로**: **`await _next(httpContext);`**를 호출하여 미들웨어 체인을 계속 진행한다. 이 줄은 전체 이름이 생성되었는지 여부와 관계없이 요청이 다음 미들웨어 구성 요소로 전달되도록 한다.
    - **중요**: **`await _next(httpContext);`** 줄은 제어를 파이프라인의 다음 미들웨어로 즉시 넘기기 때문에, 이 줄 이후의 모든 코드(예: "//after logic" 주석)는 "firstname"과 "lastname"을 모두 포함하는 요청에 대해서는 실행되지 않는다. (이는 이전 **`MyCustomMiddleware`** 예시와는 다르다. **`MyCustomMiddleware`**는 **`await next(context);`** 이후에 코드가 있었고, 이 코드는 다음 미들웨어가 완료된 후에 실행된다. **`HelloCustomMiddleware`**의 **`Invoke`** 메서드에는 **`await _next(httpContext);`** 이후에 코드가 없으므로, 이 미들웨어는 다음으로 제어를 넘기면 그 역할이 완료된다.)
- **`UseHelloCustomMiddleware` 확장**: 이 확장 메서드는 사용자 정의 미들웨어 클래스의 인스턴스화 및 사용에 대한 세부 정보를 숨겨 등록 프로세스를 단순화한다.

#### `Program.cs` (또는 `Startup.cs`): 미들웨어 사용

```csharp
// ... 다른 미들웨어 ...
app.UseMyCustomMiddleware();
app.UseHelloCustomMiddleware();
// ...
```

#### 작동 방식

요청이 도착하면 ASP.NET Core는 미들웨어 파이프라인을 탐색한다.
1. **`HelloCustomMiddleware`**에 도달하여 특정 쿼리 매개변수를 확인한다.
2. 매개변수가 존재하면 미들웨어는 개인화된 인사를 생성한다.
3. 인사를 생성하는지 여부와 관계없이 미들웨어는 **`next(context)`**를 호출하여 요청을 파이프라인의 다음 미들웨어 구성 요소로 전달한다.

#### 핵심 사항

- **단순성**: 컨벤션 기반 미들웨어는 작성하고 이해하기 쉽다.
- **제어**: 요청이 처리되고 응답이 생성되는 방식에 대한 세밀한 제어권을 가진다.
- **확장 메서드**: 확장 메서드를 사용하여 미들웨어 등록을 깔끔하고 읽기 쉽게 만든다.

---

### 미들웨어 파이프라인의 이상적인 순서

일반적으로 ASP.NET Core 미들웨어 파이프라인은 다음과 같은 이상적인 순서를 따르는 것이 권장된다. 이 순서는 애플리케이션의 성능, 보안 및 유지 관리성을 최적화하는 데 도움이 된다.

1. **예외/오류 처리 (Exception/Error Handling)**:
    
    - **목적**: 파이프라인 어디에서든 발생하는 예외를 포착하고 처리한다.
    - **예시**: **`UseExceptionHandler`**, **`UseDeveloperExceptionPage`** (개발 환경용).
    - **설명**: 가장 먼저 위치하여 애플리케이션의 어떤 부분에서 오류가 발생하더라도 즉시 처리할 수 있도록 한다.
        
2. **HTTPS 리디렉션 (HTTPS Redirection)**:
    
    - **목적**: 보안을 위해 HTTP 요청을 HTTPS로 리디렉션한다.
    - **예시**: **`UseHttpsRedirection`**.
    - **설명**: 모든 통신이 안전하게 이루어지도록 보장한다.
        
3. **정적 파일 (Static Files)**:
    
    - **목적**: 이미지, CSS, JavaScript와 같은 정적 파일을 클라이언트에 직접 제공한다.
    - **예시**: **`UseStaticFiles`**.
    - **설명**: 정적 파일 요청을 빠르게 처리하여 불필요하게 파이프라인의 나머지 부분을 통과하는 것을 방지하고 성능을 향상시킨다.
        
4. **라우팅 (Routing)**:
    
    - **목적**: 들어오는 요청을 URL을 기반으로 특정 엔드포인트에 일치시킨다.
    - **예시**: **`UseRouting`**, **`UseEndpoints`**.
    - **설명**: 요청이 어떤 컨트롤러 액션, Razor 페이지 또는 최소 API 핸들러로 가야 하는지 결정한다.
        
5. **CORS (Cross-Origin Resource Sharing)**:
    
    - **목적**: 다른 도메인에서의 안전한 교차 출처 요청을 활성화한다.
    - **예시**: **`UseCors`**.
    - **설명**: 프런트엔드 애플리케이션이 다른 도메인에서 백엔드 API를 호출할 수 있도록 허용한다.
        
6. **인증 (Authentication)**:
    
    - **목적**: 사용자 ID를 확인하고 사용자 주체(principal)를 설정한다.
    - **예시**: **`UseAuthentication`**.
    - **설명**: 요청을 보낸 사용자가 누구인지 식별한다. 이 단계에서 사용자는 아직 특정 리소스에 접근할 권한이 있는지 여부는 확인되지 않는다.
        
7. **권한 부여 (Authorization)**:
    
    - **목적**: 사용자가 특정 리소스에 접근하거나 특정 작업을 수행할 수 있는지 여부를 결정한다.
    - **예시**: **`UseAuthorization`**.
    - **설명**: 인증된 사용자가 특정 작업을 수행하거나 특정 페이지/API에 접근할 수 있는지 권한을 확인한다. **인증 다음에 권한 부여가 와야 한다.**
        
8. **사용자 정의 미들웨어 (Custom Middleware)**:
    
    - **목적**: 로깅, 기능 플래그(feature flags) 등 애플리케이션별 미들웨어 구성 요소다.
    - **설명**: 위의 핵심 미들웨어 이후, 애플리케이션의 특정 비즈니스 로직이나 추가적인 요청 처리를 담당하는 사용자 정의 미들웨어를 배치한다.
        

#### 순서에 대한 이유

- **조기 예외 처리**: 예외를 일찍 포착하면 파이프라인 아래로 전파되어 추가 문제를 일으키는 것을 방지한다.
- **보안 우선**: HTTPS 리디렉션, 인증 및 권한 부여는 애플리케이션 보안에 필수적이다.
- **성능 최적화**: 정적 파일, 응답 캐싱 및 압축은 응답 생성 프로세스를 최적화하기 위해 초기에 배치된다.
- **기반으로서의 라우팅**: 라우팅은 애플리케이션의 핵심 로직이 요청을 처리하는 방식을 결정한다.
- **유연성을 위한 CORS**: CORS는 애플리케이션이 더 넓은 범위의 클라이언트에서 사용될 수 있도록 한다.
- **사용자 정의 미들웨어**: 사용자 정의 미들웨어는 적절한 단계에서 로직을 적용하기 위해 파이프라인 내에 전략적으로 배치될 수 있다.
    
#### 유연성과 예외

이것이 일반적인 권장 순서이지만, 애플리케이션의 특정 요구 사항에 따라 예외가 있을 수 있다. 예를 들어:

- **헬스 체크**: 애플리케이션의 상태를 다른 미들웨어 구성 요소를 실행하지 않고도 빠르게 결정하기 위해 헬스 체크 미들웨어를 파이프라인의 아주 초기에 배치할 수 있다.
- **특수 미들웨어**: 일부 미들웨어 구성 요소는 제공업체에 의해 문서화된 특정 순서 요구 사항이 있을 수 있다.
    

#### 예시 (`Program.cs` 또는 `Startup.cs`):

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

if (app.Environment.IsDevelopment()) // 개발 환경에서만 개발자 예외 페이지 사용
{
    app.UseDeveloperExceptionPage();
}

app.UseHttpsRedirection(); // HTTP 요청을 HTTPS로 리디렉션
app.UseStaticFiles();      // 정적 파일 제공
app.UseRouting();          // 라우팅 활성화
app.UseAuthentication();   // 사용자 인증
app.UseAuthorization();    // 사용자 권한 부여

// ... 여러분의 사용자 정의 미들웨어 ...
// app.UseMyCustomMiddleware();

app.UseEndpoints(endpoints => // 엔드포인트 실행
{
    endpoints.MapControllers(); // 컨트롤러 라우팅 또는 MapRazorPages(), MapGet() 등
});
```

이 권장 순서를 준수하면 유지 관리, 디버깅 및 보안에 더 용이한 잘 구조화되고 효율적인 ASP.NET Core 애플리케이션을 만들 수 있다.

---

### UseWhen()

**`UseWhen()`**은 ASP.NET Core의 **`IApplicationBuilder`** 인터페이스에 있는 강력한 확장 메서드다. 조건(`predicate`)을 기반으로 미들웨어를 요청 파이프라인에 **조건부로 추가**할 수 있도록 한다. 이는 특정 미들웨어 구성 요소가 특정 조건이 충족될 때만 실행되는 동적 파이프라인을 만들 수 있음을 의미한다.

#### 구문

```csharp
app.UseWhen(
    context => /* 여기에 조건 */,          // HttpContext를 받아서 true/false 반환하는 조건
    app => /* 분기에 대한 미들웨어 구성 */ // 조건이 true일 때 실행될 미들웨어 구성
);
```

- **`context`**: 현재 요청을 나타내는 **`HttpContext`** 객체다.
- **프레디케이트 (조건)**: **`HttpContext`**를 받아 미들웨어 분기(branch)가 실행되어야 하는 경우 **`true`**를, 그렇지 않은 경우 **`false`**를 반환하는 함수다.
- **미들웨어 구성**: 조건이 **`true`**일 경우 실행되어야 하는 미들웨어 구성 요소를 구성하는 액션이다. 여기에서 **`app.Use()`**, **`app.Run()`** 또는 기타 미들웨어 등록 메서드를 사용한다.
    

#### UseWhen() 작동 방식

1. **Predicate 평가**: 요청이 들어오면 **`UseWhen()`** 메서드는 먼저 **`HttpContext`**에 대해 Predicate 함수를 평가한다.
	1. 프레디케이트(Predicate)는 간단히 말해 **'참(True) 또는 거짓(False)을 반환하는 조건'**이라고 생각하면 된다.
2. **분기(true인 경우)**: 프레디케이트가 **`true`**를 반환하면, 구성 액션에 지정된 미들웨어 분기가 실행된다. 요청은 이 분기를 통해 흐르며, 잠재적으로 수정되거나 응답을 생성한다.
3. **주 파이프라인 재합류**: 분기가 실행된 후(또는 프레디케이트가 **`false`**여서 건너뛰어진 경우), 요청 흐름은 주 파이프라인에 다시 합류하여 **`UseWhen()`** 호출 이후에 등록된 다음 미들웨어 구성 요소로 계속 진행된다.
    

#### 코드 예시 설명

```csharp
app.UseWhen(
    context => context.Request.Query.ContainsKey("username"), // "username" 쿼리 파라미터가 있는지 확인
    app => {
        app.Use(async (context, next) =>
        {
            await context.Response.WriteAsync("Hello from Middleware branch\n"); // 분기 미들웨어
            await next(); // 분기 내의 다음 미들웨어 또는 주 파이프라인으로 제어 반환
        });
    });

app.Run(async context =>
{
    await context.Response.WriteAsync("Hello from middleware at main chain"); // 주 파이프라인의 최종 미들웨어
});
```

- **조건**: 프레디케이트 **`context.Request.Query.ContainsKey("username")`**는 쿼리 문자열에 "username"이라는 매개변수가 포함되어 있는지 확인한다.
- **분기 미들웨어**: "username" 매개변수가 있으면 분기 미들웨어가 실행된다. 이 미들웨어는 "Hello from Middleware branch"를 응답에 작성한 다음 **`next`**를 호출하여 파이프라인의 나머지 부분이 계속되도록 한다. **여기서 `await next();`는 `UseWhen` 블록 내의 다음 미들웨어 (만약 있다면) 또는 `UseWhen` 블록 뒤의 메인 파이프라인으로 제어를 돌려준다.**
- **주 파이프라인**: 최종 **`app.Run`** 미들웨어는 주 파이프라인의 일부다. "Hello from middleware at main chain"을 응답에 작성한다.
    

#### 출력

요청에 "username" 쿼리 매개변수가 포함된 경우(예: `/path?username=John`), 출력은 다음과 같다.

```
Hello from Middleware branch
Hello from middleware at main chain
```

요청에 "username" 매개변수가 포함되지 않은 경우(예: `/path`), 출력은 다음과 같다.

```
Hello from middleware at main chain
```

#### `UseWhen()`을 언제 사용해야 하는가?

- **조건부 기능**: 요청에 따라 특정 기능을 활성화하거나 비활성화한다(예: 특정 사용자에게만 로깅, 쿼리 매개변수에 따른 캐싱 규칙 적용).
- **동적 파이프라인**: 다른 요청에 따라 조정되는 파이프라인을 생성한다(예: 특정 경로에 대해 다른 인증 미들웨어).
- **A/B 테스팅**: 실험을 위해 사용자 하위 집합을 대체 미들웨어 분기를 통해 라우팅한다.
- **디버깅 및 진단**: 개발 환경에서만 진단 미들웨어를 적용한다.
    

---

### 기억해야 할 핵심 포인트

#### 개념적 이해:

- **파이프라인**: 미들웨어는 HTTP 요청 및 응답을 위한 파이프라인을 형성한다. 각 구성 요소는 흐름을 검사, 수정 또는 종료할 수 있다.
- **순서가 중요**: 미들웨어는 등록된 순서대로 실행된다. 순서에 대해 신중하게 생각해야 한다.
- **미들웨어 유형**:
    
    - **내장**: ASP.NET Core는 인증, 라우팅, 정적 파일 등을 위한 미들웨어를 제공한다.
    - **사용자 정의**: 애플리케이션에 특정 로직을 추가하기 위해 직접 만들 수 있다.
        

#### `app.Use` 대 `app.Run`

- **`app.Use`**: 비-종료 미들웨어용이다. **`next`**를 호출하여 다음 구성 요소로 제어를 전달한다.
- **`app.Run`**: 종료 미들웨어용이다. 파이프라인을 종료하고 응답을 생성한다.
    

#### 사용자 정의 미들웨어:

- **두 가지 방법**:
    
    - **컨벤션 기반(Conventional)**: 클래스 기반이며, **`Invoke`** 메서드와 생성자 주입을 사용한다.
    - **팩토리 기반(Factory-Based)**: 미들웨어 인스턴스를 생성하는 델리게이트를 사용한다. (이 문서에서는 컨벤션 기반만 다루고 있다.)
- **장점**: 로직을 캡슐화하고, 코드 구성을 개선하며, 애플리케이션의 동작을 맞춤 설정할 수 있도록 한다.

#### 권장 순서

1. 예외 처리
2. HTTPS 리디렉션
3. 정적 파일
4. 라우팅
5. CORS
6. 인증
7. 권한 부여
8. 사용자 정의 미들웨어
9. MVC/Razor Pages/Minimal APIs
    

#### 보너스 포인트

- **단락(Short-Circuiting)**: 미들웨어는 **`next`**를 호출하지 않고 일찍 응답을 반환하도록 선택할 수 있다.
- **`UseWhen`**: 요청 기준에 따라 미들웨어 분기를 조건부로 추가한다.
- **미들웨어 순서 유연성**: 권장 순서의 이유를 이해하고, 애플리케이션의 특정 요구 사항에 따라 언제 순서를 벗어날 수 있는지 알아야 한다.