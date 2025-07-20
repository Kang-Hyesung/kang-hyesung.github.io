---
title: Middleware2
date: 2025-07-20 21:04 +0900
author: hyesung
---
ASP.NET Core 미들웨어 체인은 요청 처리 흐름을 파이프라인 형태로 구성해, 필요에 따라 다음 미들웨어로 넘기거나 중단해 제어하는 구조다.

1. 미들웨어 체인
   요청이 들어오면 등록된 순서대로 각 미들웨어가 실행된다. 각 미들웨어는 자신의 단일 책임(로깅, 인증, HTTPS 리다이렉션 등)을 수행한 뒤, `next` 호출로 다음 미들웨어로 넘기거나 호출을 생략해 체인을 종료할 수 있다.

2. `app.Use` vs `app.Run`

   * `app.Use(...)`

     * 인수 2개: `HttpContext`, `RequestDelegate next`
     * `next(context)` 호출 시 체인을 계속 이어간다.
     * 호출 생략 시 해당 위치에서 즉시 응답을 반환하며 이후 미들웨어는 실행되지 않는다.
   * `app.Run(...)`

     * 인수 1개: `HttpContext`
     * `next` 매개변수가 없어 항상 체인을 종료(short‑circuit)한다.

3. `RequestDelegate` (`next`) 역할
   다음 미들웨어에 현재 `HttpContext`를 전달하는 대리자다. 호출 이후에 뒤따르는 코드(후행 로직)가 체인이 모두 돌아온 뒤 실행된다.

4. 종료(terminating) 미들웨어
   `app.Run` 또는 `app.Use`에서 `next`를 호출하지 않는 경우, 해당 미들웨어가 terminating middleware가 되어 즉시 응답을 반환한다.

5. 커스텀 미들웨어 권장
   미들웨어 로직이 복잡해지면 별도 클래스(파일)로 분리해 관리해야 유지보수가 용이하다.

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// ① app.Use: next 호출로 체인 이어가기, 생략 시 체인 종료
app.Use(async (HttpContext context, RequestDelegate next) =>
{
    await context.Response.WriteAsync("hello");
    await next(context);  // 다음 미들웨어로 전달
    // (선택) 후행 로직 실행
});

// ② app.Run: 항상 체인 종료 (terminating middleware)
app.Run(async context =>
{
    await context.Response.WriteAsync("hello 2");
});

app.Run();  // 서버 실행
```
