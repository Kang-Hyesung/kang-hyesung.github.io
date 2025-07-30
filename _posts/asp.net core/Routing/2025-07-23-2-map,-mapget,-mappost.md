---
title: "[2] Map, MapGet, MapPost"
date: 2025-07-23 23:36 +0900
author: hyesung
---
## 엔드포인트와 라우팅의 기본

ASP.NET Core에서 라우팅은 들어오는 HTTP 요청의 URL(경로)을 분석하여 어떤 코드를 실행할지 결정하는 핵심 과정이다. 이때 실행되도록 지정된 코드 조각을 **엔드포인트(Endpoint)** 라고 한다. 본질적으로 엔드포인트는 라우팅에 의해 트리거되는 특수한 형태의 미들웨어다. 즉, 특정 URL 패턴과 요청 처리 로직(미들웨어)을 서로 매핑하는 것이다. 요청 URL이 미리 정의된 패턴과 일치하면, 매핑된 엔드포인트가 실행되어 요청을 처리한다.

`UseEndpoints` 메서드 내에서 `Map`, `MapGet`, `MapPost`와 같은 `Map`으로 시작하는 다양한 확장 메서드를 사용하여 이 매핑을 손쉽게 정의할 수 있다. 이 메서드들은 `Program.cs` 파일에서 `app.UseRouting()` 호출 뒤에 위치해야 올바르게 동작한다.

## endpoints.Map(): 모든 HTTP 메서드 처리

가장 기본적인 엔드포인트 매핑 메서드는 `Map`이다. 이 메서드는 URL 경로는 일치시키지만 HTTP 메서드(GET, POST 등)의 종류는 구분하지 않는다. 따라서 특정 URL로 들어오는 모든 종류의 HTTP 요청을 동일한 로직으로 처리하고 싶을 때 유용하다.

```
// 'map1' 경로로 들어오는 모든 요청을 처리
endpoints.Map("map1", async context =>
{
    await context.Response.WriteAsync("In Map 1");
});

// 'map2' 경로로 들어오는 모든 요청을 처리
endpoints.Map("map2", async context =>
{
    await context.Response.WriteAsync("In Map 2");
});
```

위 코드에서 `/map1` URL로 요청을 보내면 "In Map 1"이라는 응답을 받게 된다. 이는 브라우저 주소창을 통해 보내는 `GET` 요청이든, HTML form을 통해 보내는 `POST` 요청이든, API 클라이언트를 통해 보내는 `PUT` 또는 `DELETE` 요청이든 상관없이 동일하게 동작한다.

### 일치하지 않는 URL 처리

만약 `map1`이나 `map2`가 아닌 다른 URL(예: `/` 또는 `/home`)로 요청하면 어떻게 될까? 라우팅 시스템이 요청과 일치하는 엔드포인트를 찾지 못하므로, 요청은 파이프라인의 다음 미들웨어로 전달된다. 만약 더 이상 처리할 미들웨어가 없다면, 프레임워크는 기본적으로 클라이언트에게 **404 Not Found** 오류를 반환한다.

이러한 경우를 명시적으로 처리하기 위해, 모든 엔드포인트 매핑 뒤에 `app.Run()`과 같은 터미널 미들웨어를 추가하여 "기본" 또는 "catch-all" 응답을 정의할 수 있다.

```csharp
app.Run(async context => {
  await context.Response.WriteAsync($"Request received at {context.Request.Path}");
});
```
UseEndpoints에 의해 요청이 성공적으로 처리되면, 해당 엔드포인트는 응답을 생성하고 요청 처리를 종료한다. 이 경우 요청은 파이프라인의 뒷부분에 있는 `app.Run` 미들웨어로 전달되지 않는다. 이를 **단락(short-circuiting)** 이라고 하며, 불필요한 코드 실행을 막아 성능을 향상시킨다. 즉, 요청 처리 흐름은 다음과 같다

`요청 -> UseRouting -> UseEndpoints (일치하는 엔드포인트 발견) -> 엔드포인트 실행 및 응답 (파이프라인 종료)`

만약 일치하는 엔드포인트가 없다면 흐름은 계속된다
`... UseEndpoints (일치하는 엔드포인트 없음) -> app.Run 실행`

## MapGet & MapPost: 특정 HTTP 메서드 처리

실제 애플리케이션에서는 같은 URL이라도 HTTP 메서드에 따라 다른 동작을 수행해야 하는 경우가 훨씬 많다. 예를 들어, `/board`라는 URL에 `GET` 요청을 보내면 게시글 목록을 조회하고, `POST` 요청을 보내면 새로운 게시글을 등록하는 기능이 필요하다. `Map` 메서드만으로는 이를 구현할 수 없다.

이럴 때 `MapGet`, `MapPost`, `MapPut`, `MapDelete` 등 특정 HTTP 메서드에만 응답하는 전용 매핑 메서드를 사용한다. 이 메서드들은 코드의 의도를 명확하게 드러내고, RESTful 원칙에 맞는 API를 설계하는 데 필수적이다.

-   `MapGet`: `GET` 요청에만 응답한다. 데이터 조회를 목적으로 하며, 멱등성(idempotent)을 가진다.
-   `MapPost`: `POST` 요청에만 응답한다. 새로운 리소스 생성을 목적으로 한다.

### 최종 예제 코드

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// 1. 라우팅 기능 활성화
app.UseRouting();

// 2. 엔드포인트 정의
app.UseEndpoints(endpoints =>
{
    // GET /map1 요청만 처리
    endpoints.MapGet("map1", async (context) => {
        await context.Response.WriteAsync("In Map 1 (GET)");
    });

    // POST /map2 요청만 처리
    endpoints.MapPost("map2", async (context) => {
        await context.Response.WriteAsync("In Map 2 (POST)");
    });
});

// 3. 위 엔드포인트와 일치하지 않는 모든 요청 처리
app.Run(async context => {
    await context.Response.WriteAsync($"Request received at {context.Request.Path}");
});

app.Run();
```

- `GET` /map1: "In Map 1 (GET)" 응답을 성공적으로 받는다.
- `POST` /map1: `MapGet`으로 정의했기 때문에 `POST` 요청을 처리할 수 없다. 이때 ASP.NET Core는 `404 Not Found`가 아닌 **405 Method Not Allowed** 상태 코드를 응답한다. 이는 "해당 URL은 존재하지만, 당신이 사용한 HTTP 메서드는 허용되지 않는다"는 훨씬 더 구체적인 정보를 클라이언트에게 제공한다.
- `GET` /map2: 마찬가지로 `MapPost`로 정의했으므로 `GET` 요청에 대해 **405 Method Not Allowed** 오류가 발생한다.
- `POST` /map2: "In Map 2 (POST)" 응답을 성공적으로 받는다.

이처럼 `MapGet`, `MapPost` 등의 메서드를 사용하면 특정 URL과 HTTP 메서드의 조합에 대해 명확하고 구체적인 엔드포인트를 구성할 수 있다. 이는 RESTful API를 설계할 때 매우 유용하며, 코드의 가독성과 유지보수성을 크게 향상시킨다.