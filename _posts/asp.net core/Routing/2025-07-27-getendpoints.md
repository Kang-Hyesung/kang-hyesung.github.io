---
title: "[3] GetEndpoints"
date: 2025-07-27 22:13 +0900
author: hyesung
---
ASP.NET Core 애플리케이션에서 요청이 처리되는 방식의 중심에는 **미들웨어 파이프라인(Middleware Pipeline)**이 있고, 라우팅은 이 파이프라인의 핵심적인 부분이다. 이 글에서는  `UseRouting()`과 `UseEndpoints()`가 각각 어떤 역할을 하며, 이들 사이에서 `HttpContext.GetEndpoint()`를 통해 어떻게 엔드포인트(Endpoint) 정보를 확인하는지 알아본다.

### 🤔 핵심 개념: 선택과 실행의 분리

ASP.NET Core 라우팅의 가장 중요한 특징은 **엔드포인트를 선택하는 단계**와 **선택된 엔드포인트를 실행하는 단계**가 분리되어 있다는 점이다.

1. **`app.UseRouting()` - 엔드포인트 선택자**
    
    - 이 미들웨어는 들어오는 요청(Request)의 URL, HTTP 메서드 등을 분석한다.
    - 그리고 코드에 정의된 여러 엔드포인트(`MapGet`, `MapPost` 등) 중에서 어떤 것을 실행해야 할지 **결정하고 선택**한다.
    - **중요:** 이 단계에서는 코드를 _실행하지 않고_ 어떤 코드를 실행할지만 결정하여 `HttpContext`에 저장한다. 마치 식당에서 메뉴판을 보고 주문할 음식을 고르는 것과 같다.
        
2. **`app.UseEndpoints()` - 엔드포인트 실행자**
    
    - `UseRouting()`에 의해 선택된 엔드포인트의 코드를 **실제로 실행**하는 역할을 한다.
    - 선택된 엔드포인트가 없다면 아무것도 실행하지 않는다. 식당에서 주문한 음식을 요리사가 직접 만들어주는 과정에 비유할 수 있다.

이 두 과정 사이에서, 즉 "음식을 고른 직후, 요리가 시작되기 전"에 어떤 음식이 선택되었는지 엿보는 것은 `HttpContext.GetEndpoint()` 메서드를 사용하면 가능하다.

---

### 💻 코드로 확인

아래 코드는 `UseRouting()` 전과 후에 각각 미들웨어를 배치하여 `GetEndpoint()` 메서드의 반환 값이 어떻게 달라지는지 보여주는 예제다.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// --- 1. 'UseRouting' 호출 전 미들웨어 ---
// 이 시점에서는 아직 엔드포인트가 선택되지 않았다.
app.Use(async (context, next) =>
{
    // GetEndpoint()를 호출해도 아직 라우팅이 결정되지 않았으므로 null을 반환한다.
    Microsoft.AspNetCore.Http.Endpoint? endPoint = context.GetEndpoint();

    // endpoint가 null이 아닐 때만 DisplayName을 출력하도록 조건 추가
    if (endPoint != null)
    {
        await context.Response.WriteAsync($"[Before Routing] Endpoint: {endPoint.DisplayName}\n");
    }

    // 다음 미들웨어로 요청을 전달한다.
    await next(context);
});

// --- 2. 라우팅 미들웨어 활성화 ---
// 이 메서드가 실행되면서 들어온 요청에 맞는 엔드포인트를 '선택'한다.
app.UseRouting();

// --- 3. 'UseRouting' 호출 후 미들웨어 ---
// 이제 엔드포인트가 선택되었으므로 정보를 확인할 수 있다.
app.Use(async (context, next) =>
{
    // UseRouting()이 실행된 후이므로, 일치하는 엔드포인트 객체를 가져올 수 있다.
    Microsoft.AspNetCore.Http.Endpoint? endPoint = context.GetEndpoint();

    if (endPoint != null)
    {
        // 엔드포인트의 DisplayName과 같은 상세 정보를 응답에 출력한다.
        await context.Response.WriteAsync($"[After Routing] Endpoint: {endPoint.DisplayName}\n");
    }

    await next(context);
});

// --- 4. 엔드포인트 실행 미들웨어 ---
// '선택된' 엔드포인트를 '실행'한다.
app.UseEndpoints(endpoints =>
{
    // GET /map1 요청에 대한 엔드포인트를 정의한다.
    endpoints.MapGet("map1", async (context) => {
        await context.Response.WriteAsync("Response from Map 1\n");
    });

    // POST /map2 요청에 대한 엔드포인트를 정의한다.
    endpoints.MapPost("map2", async (context) => {
        await context.Response.WriteAsync("Response from Map 2\n");
    });
});

// 일치하는 엔드포인트가 없을 경우 실행될 폴백(Fallback) 미들웨어
app.Run(async context => {
    await context.Response.WriteAsync($"No endpoint found for {context.Request.Path}\n");
});

app.Run();
```

---

### ✨ 실행 흐름 분석

#### 🤔 `UseRouting`은 한번만 실행되는거 아닌가??

이 코드를 보면 "서버가 켜질 때 `UseRouting`이 이미 실행됐을 텐데, 왜 첫 번째 미들웨어에서 엔드포인트를 못 찾는 거지?" 라는 생각이 들었는데. 이는 **애플리케이션 시작**과 **개별 요청 처리**를 구분해야 이해할 수 있다.

- 🏭 1. 애플리케이션 시작 (파이프라인 구성)
  app.Run()이 호출되기 전까지의 코드는 실제 요청을 처리하는 게 아니라, 요청 처리 파이프라인이라는 컨베이어 벨트를 설계하고 조립하는 과정이다. app.UseRouting() 같은 코드는 "여기에 라우팅 부품을 놓겠다"고 선언하는 것과 같다. app.Run()은 이 컨베이어 벨트를 가동시키는 스위치다.
    
- 🚗 2. HTTP 요청 처리 (파이프라인 실행)
  사용자가 /map1 같은 URL을 호출하면, 이 요청은 새로운 상자처럼 우리가 조립한 컨베이어 벨트의 맨 처음부터 여행을 시작한다. 모든 요청은 이 벨트를 처음부터 끝까지 통과하며 각 부품(미들웨어)을 순서대로 거친다. 이전 요청이 어디까지 갔는지는 다음 요청과 아무 상관이 없다.
    

#### 🗺️ `/map1` 요청의 실제 여행 경로

위 개념을 바탕으로 `/map1` GET 요청의 흐름을 다시 따라가 보자.

1. **요청 시작**: 요청이 컨베이어 벨트의 시작점에 놓인다.
2. **첫 번째 미들웨어 통과**: 이 시점에서 상자(`HttpContext`)는 비어있다. 라우팅 부품(`UseRouting`)을 아직 안 거쳤으므로 `context.GetEndpoint()`는 당연히 `null`이다.
3. **`app.UseRouting()` 통과**: 라우팅 부품이 상자를 열고, 요청이 `/map1`인 것을 확인한 뒤 'map1 엔드포인트'라는 스티커를 붙인다.
4. **두 번째 미들웨어 통과**: 이제 상자에 'map1 엔드포인트' 스티커가 붙어있으므로, `context.GetEndpoint()`는 해당 엔드포인트 정보를 성공적으로 반환한다.
5. **`app.UseEndpoints()` 통과**: 'map1 엔드포인트' 스티커를 확인하고, 그에 맞는 실제 작업(`Response from Map 1` 출력)을 수행한다.

최종적으로 브라우저에는 아래와 같은 결과가 나타난다.

```
[After Routing] Endpoint: HTTP: GET /map1
Response from Map 1
```

---

### ✅ 실용적인 사용 사례

엔드포인트가 실행되기 _전에_ 어떤 엔드포인트가 선택되었는지 아는 것은 매우 유용하다. 예를 들어 다음과 같은 시나리오에서 활용할 수 있다.

- **동적 로깅(Dynamic Logging)**: 특정 엔드포인트 그룹(예: `/api/`로 시작하는 모든 엔드포인트)에 대해서만 상세한 요청/응답 로그를 남기고 싶을 때, `UseRouting`과 `UseEndpoints` 사이의 미들웨어에서 엔드포인트 정보를 확인하고 조건부로 로깅을 수행할 수 있다.
- **커스텀 권한 부여(Custom Authorization)**: 특정 엔드포인트에 대한 접근을 동적으로 제어해야 할 때, 선택된 엔드포인트의 메타데이터(Metadata)를 읽어와 추가적인 권한 검사를 수행할 수 있다.
    

---

### 요약

- `app.UseRouting()`은 요청에 맞는 엔드포인트를 **선택**한다.
- `app.UseEndpoints()`은 선택된 엔드포인트를 **실행**한다.
- 미들웨어 파이프라인은 서버 시작 시 **한 번만 구성**되고, 모든 HTTP 요청은 이 파이프라인을 **매번 처음부터 끝까지** 통과한다.
- 이 때문에 `UseRouting`보다 앞에 있는 미들웨어는 해당 요청에 대한 엔드포인트 정보를 알 수 없다.