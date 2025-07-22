---
title: UseWhen 을 이용한 조건부 실행
date: 2025-07-22 17:04 +0900
author: hyesung
---
ASP.NET Core는 웹 애플리케이션 요청 파이프라인에 미들웨어를 연결하는 데 `Use` 메서드를 쓴다. 이와 비슷하게 `UseWhen`이라는 다른 메서드가 있고, 이건 특정 조건이 참일 때 미들웨어의 **분기(branch)**를 실행하는 데 쓴다.

### Use 대 UseWhen
- **`Use`**: `Use`는 모든 요청에 미들웨어를 애플리케이션 요청 파이프라인에 연결한다.
- **`UseWhen`**: `UseWhen`은 특정 조건이 충족될 때만 미들웨어의 특정 분기를 실행한다.

### `UseWhen`의 작동 방식

`UseWhen`은 다음 흐름으로 작동한다:
1. **초기 미들웨어 실행**: 요청이 들어오면 요청 파이프라인의 첫 미들웨어가 실행된다.
2. **`UseWhen` 조건 확인**: 그 후에 `UseWhen`에 정의된 조건을 확인한다. 이 조건은 요청의 헤더 값, 쿼리 문자열, 요청 메서드 등 요청의 어떤 세부 사항이든 확인할 수 있다.
3. **조건부 미들웨어 분기 실행**:
    - **조건이 참일 경우**: `UseWhen` 안에 정의된 미들웨어 분기(즉, 일련의 미들웨어)가 실행된다. 이 분기가 완료된 후에는 일반 미들웨어 **주 체인(main chain)**으로 계속 진행한다.
    - **조건이 거짓일 경우**: 해당 미들웨어 분기는 건드리지 않고, 요청은 일반 미들웨어 주 체인으로 바로 계속 진행한다.

**주 체인(main chain)**은 모든 요청에 대해 공통으로 실행되어야 하는 미들웨어의 실제 컬렉션을 말한다.
간단히 말해, `UseWhen`으로 정의된 미들웨어 분기는 특정 조건이 참일 때만 실행되고, 그렇지 않으면 실행되지 않는다.

---

### 예제

#### `Program.cs` 코드

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseWhen(
    // 첫 번째 인자: 불리언 값을 반환하는 람다 표현식 (조건)
    context => context.Request.Query.ContainsKey("username"),
    // 두 번째 인자: 조건이 참일 때 실행할 미들웨어 분기
    app => {
        app.Use(async (context, next) =>
        {
            await context.Response.WriteAsync("Hello from Middleware branch");
            await next(); // 다음 미들웨어로 제어를 전달한다.
        });
    });

// 위 UseWhen 조건과 관계없이 실행되는 미들웨어 (주 체인)
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello from middleware at main chain");
});
```

#### 코드 설명
1. **`app.UseWhen`**:
    - **조건 (`context => context.Request.Query.ContainsKey("username")`)**: 이 람다 표현식은 요청 쿼리 문자열에 'username'이라는 키가 있는지 확인한다. 이 조건이 참일 경우에만 두 번째 인자로 전달된 미들웨어 분기가 실행된다.
    - **미들웨어 분기 (`app => { ... }`)**: 이 람다 표현식은 `UseWhen` 조건이 참일 때 실행될 미들웨어를 정의한다. 여기서는 "Hello from Middleware branch"라는 응답을 작성하는 간단한 미들웨어가 있다. `await next()`를 호출해서 분기 안의 다음 미들웨어로 제어를 전달한다.
2. **`app.Run`**:
    - 이 `app.Run` 미들웨어는 `UseWhen` 조건의 참/거짓 여부와 관계없이 항상 실행되는 주 체인의 미들웨어다. "Hello from middleware at main chain"이라는 응답을 작성한다.

#### 실행 결과

- 일반 요청 (예: http://localhost:5000/):
    'username' 쿼리 문자열이 없어 UseWhen 조건이 거짓이다. 그래서 미들웨어 분기는 실행되지 않고, "Hello from middleware at main chain"이라는 응답만 표시된다.
- 'username' 쿼리 문자열을 포함하는 요청 (예: http://localhost:5000/?username=harsha):
    'username' 쿼리 문자열이 있어 UseWhen 조건이 참이다. 미들웨어 분기가 실행되어 "Hello from Middleware branch"를 먼저 쓰고, 그 후 주 체인으로 제어가 전달되어 "Hello from middleware at main chain"이 이어서 쓰인다. 따라서 최종 출력은 "Hello from Middleware branchHello from middleware at main chain"이 된다.

---
### 언제 UseWhen 을 써야 할까?

`UseWhen`은 특정 조건이 참일 때만 일련의 미들웨어를 실행하고 싶을 때 유용하다. 예를 들면:
- **인증 확인**: 요청 헤더에 인증 토큰이 있을 때만 특정 인증 미들웨어를 실행할 수 있다.
- **특정 경로 처리**: 특정 경로 요청에 대해서만 추가 로깅 또는 유효성 검사 미들웨어를 적용할 수 있다.
- **사용자 역할 기반 로직**: 특정 사용자 역할이 있을 때만 관리자 패널 관련 미들웨어를 실행할 수 있다.

요청의 특정 조건에 따라 다른 미들웨어 파이프라인을 동적으로 구성해야 할 때 `UseWhen`을 효과적으로 활용할 수 있다.