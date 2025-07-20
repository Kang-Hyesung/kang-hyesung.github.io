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