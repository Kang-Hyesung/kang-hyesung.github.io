---
title: "[1] Middleware 개요"
author: hyesung
date: 2025-07-16 12:00:00 +0900
pin: false
math: false
mermaid: false
---

```csharp
var builder = WebApplication.CreateBuilder(args);
// 기본적으로 빌드 메서드를 호출하면 애플리케이션 빌더 객체를 얻는다.

var app = builder.Build();
// 이 애플리케이션 빌더 객체(app 객체)는 미들웨어를 활성화하거나 생성하는데 사용된다.
// 미들웨어는 원하는 실행 순서대로 생성할 수 있다.
// 미들웨어를 생성하는 방법 중 하나는 Run 메소드를 사용하는 것이다.

// 이 람다 표현식은 요청을 수신했을 때만 실행되며, 애플리케이션 시작 시에는 실행되지 않는다.
// 요청을 받을 때만 실행되며, 이 람다 표현식은 context라는 인수를 받아야 한다.
// 선택적으로 데이터 타입을 지정할 수 있다.

// 여기서 context는 요청(Request), 응답(Response), 세션(Session) 등
// 응답을 제공하고 데이터를 처리하는 데 필요한 여러 속성을 포함하는 객체라는 의미다.
app.Run(async (HttpContext context) =>
{
    // WriteAsync 메서드를 사용하므로 await 키워드를 사용해야 한다.
    // 이 람다식에서 최소 한 번 이상 await를 사용하므로 전체 람다식을 async로 선언해야 한다.
    // 즉, 이 문장이 완전히 실행될 때까지 실행 흐름이 대기해야 한다.
    // 이 영역의 이후 문장들은 아래 문장이 완료될 때까지 대기해야 한다.
    // 하지만 app.Run 미들웨어는 여기서 응답을 보내면 파이프라인을 종료하고,
    // 그동안 서버는 다른 브라우저 요청을 병렬로 처리할 수 있다.
    await context.Response.WriteAsync("hello");
});

// 아래 식을 추가하고 빌드해보면,
// 첫 번째 미들웨어 실행 후 두 번째 미들웨어는 실행되지 않는다.
// app.Run 메소드는 요청을 다음 미들웨어로 전달하지 않는다.
// 그러나 미들웨어 개념의 기본 설계 목표는
// “미들웨어가 요청을 다음 미들웨어에게 전달할 수 있어야 한다”는 것이다.
app.Run(async (HttpContext context) =>
{
    await context.Response.WriteAsync("hello 2");
});

// app.Run은 요청을 다음 미들웨어로 전달하지 않는,
// 종료용(terminal) 미들웨어를 생성하는 데 사용된다.
app.Run();
```

---

### 핵심 요약

* **`WebApplication.CreateBuilder(args)`**

  * 애플리케이션 빌더 객체 생성
  * 초기 설정, DI(의존성 주입), 로깅, 설정 파일 읽기 등 준비

* **`builder.Build()` → `app`**

  * `WebApplication` 객체 생성
  * 이 시점부터 `app`을 통해 미들웨어 파이프라인 구성 가능

* **`app.Run(async (HttpContext context) => { … })`**

  * **터미널(종료) 미들웨어** 등록
  * 람다식은 **요청을 수신했을 때만** 실행
  * `await context.Response.WriteAsync(...)` 로 응답 전송
  * **파이프라인을 다음 미들웨어로 전달하지 않고 종료**

* **`HttpContext context`**

  * 요청(Request), 응답(Response), 세션(Session), 사용자 정보(User) 등
  * 처리에 필요한 모든 속성 포함
  * 비동기 작업 시 `async/await` 사용 필수

* **파이프라인 흐름**

  1. 첫 번째 `app.Run` 실행 → “hello” 응답 후 즉시 종료
  2. 두 번째 `app.Run` 절대 호출되지 않음
  3. 마지막 `app.Run()` 호출만으로도 터미널 미들웨어 역할 수행

* **다중 미들웨어 연결**

  * 요청을 다음 미들웨어로 넘기려면 `app.Use(...)` 사용

  ```csharp
  app.Use(async (context, next) =>
  {
      // 전처리 작업
      await next();   // 다음 미들웨어 실행
      // 후처리 작업
  });
  ```

  * `Use`는 `next()` 호출로 파이프라인 연결, `Run`은 항상 **마지막**에 한 번만 사용 권장

