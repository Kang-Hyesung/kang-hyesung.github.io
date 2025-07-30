---
title: "[7] Route 제약조건"
date: 2025-07-30 21:01 +0900
author: hyesung
---
### 개요 🚀

ASP.NET Core에서 **라우팅(Routing)** 은 들어온 요청을 올바른 코드 로직으로 안내하는 내비게이션 시스템과 같다. 때로는 이 내비게이션에 더욱 정교한 규칙을 적용하여, 특정 형태의 주소로 온 요청만 받아들이고 싶을 때가 있다. 예를 들어, 제품 ID는 반드시 숫자여야 하고, 사용자 이름은 특정 길이를 만족해야 하며, 보고서의 연도는 특정 범위 안에 있어야 한다는 등의 규칙이다.

이러한 정교한 URL 규칙을 적용할 수 있게 해주는 강력한 도구가 바로 **라우팅 제약 조건(Route Constraints)** 이다. 이 글에서는 라우팅 제약 조건의 기본 개념부터 `guid`, 길이, 범위, 정규 표현식을 활용한 고급 기법까지 전부 다룬다. 더 나아가, 제약 조건을 언제 사용하고 언제 사용하지 말아야 하는지에 대한 실용적인 고찰까지 깊이 있게 살펴본다.

---

### 1. 라우팅 제약 조건, 왜 필요한가?

기본적으로 ASP.NET Core의 라우트 매개변수는 모든 유형의 값을 허용한다. `/products/{id}` 라는 라우트는 `products/123` (숫자) 뿐만 아니라 `products/abc` (문자열) 요청까지 모두 받아들인다. 이는 `id`를 숫자로 기대하는 로직에서 런타임 오류를 발생시킬 수 있는 잠재적 위험이다.

**라우팅 제약 조건**은 이러한 문제를 해결한다. 라우트가 요청에 응답하기 전에 URL의 매개변수 값이 특정 조건을 만족하는지 먼저 검사한다. 조건에 맞지 않으면 해당 라우트는 요청을 처리하지 않고, ASP.NET Core는 다른 일치하는 라우트를 찾거나 최종적으로 404 (Not Found)와 같은 응답을 반환한다. 이를 통해 API의 입구에서부터 잘못된 요청을 걸러내어 안정성을 크게 높일 수 있다.

---

### 2. 기본 제약 조건: 타입(Type) 검증

가장 기본적이고 많이 사용되는 제약 조건은 매개변수의 데이터 타입을 검증하는 것이다.

- **`int`, `long`, `decimal`, `double`**: 숫자 형식의 값을 강제한다.
- **`bool`**: `true` 또는 `false` 값만 허용한다.
- **`datetime`**: `2025-07-30`과 같이 유효한 날짜 및 시간 형식만 허용한다.
- **`guid`**: 데이터베이스 기본 키 등으로 자주 사용되는 전역 고유 식별자(Globally Unique Identifier) 값만 허용한다. GUID는 `ca761232-ed42-11ce-bacd-00aa0057b223`와 같은 128비트 16진수 형태를 가진다.

**C# 코드 예시 (`guid` 제약 조건)**

```csharp
// URL 예시: /cities/ca761232-ed42-11ce-bacd-00aa0057b223
endpoints.Map("cities/{cityid:guid}", async context =>
{
    // 'cityid'는 항상 유효한 GUID 값임이 보장된다.
    Guid cityId = Guid.Parse(Convert.ToString(context.Request.RouteValues["cityid"])!);
    await context.Response.WriteAsync($"City information - {cityId}");
});
```

> **💡 팁:** Visual Studio의 `도구 > GUID 만들기` 메뉴를 사용하면 테스트용 GUID를 쉽게 생성할 수 있다.

---

### 3. 고급 제약 조건: 길이, 범위, 그리고 패턴(Pattern) 검증

타입 검증을 넘어 더 복잡한 규칙이 필요할 때 사용하는 고급 제약 조건들을 알아보자. 여러 제약 조건은 콜론(`:`)으로 연결하여 함께 사용할 수 있다.

#### 길이 제약 조건 (Length Constraints)

문자열 매개변수의 길이를 제한한다.

- `minlength(value)`: 최소 길이를 지정한다.
- `maxlength(value)`: 최대 길이를 지정한다.
- `length(min, max)`: 최소 길이와 최대 길이를 동시에 지정하는 약식 표현이다.
    

**C# 코드 예시**

```csharp
// EmployeeName은 알파벳(alpha)으로만 구성되어야 하며, 길이는 4~7자(length(4,7))여야 한다.
// 기본값으로 "harsha"를 가진다.
endpoints.Map("employee/profile/{EmployeeName:alpha:length(4,7)=harsha}", async context =>
{
    string? employeeName = Convert.ToString(context.Request.RouteValues["employeename"]);
    await context.Response.WriteAsync($"In Employee profile - {employeeName}");
});
```

#### 범위 제약 조건 (Range Constraints)

숫자 매개변수의 값 범위를 제한한다.

- `min(value)`: 최솟값을 지정한다.
- `max(value)`: 최댓값을 지정한다.
- `range(min, max)`: 최솟값과 최댓값을 동시에 지정하는 약식 표현이다.
    

**C# 코드 예시**

```csharp
// id는 정수(int)여야 하며, 1~1000 사이의 값(range(1,1000))이어야 한다.
// '?'는 이 매개변수가 선택적(optional)임을 의미한다.
endpoints.Map("products/details/{id:int:range(1,1000)?}", async context => {
    if (context.Request.RouteValues.ContainsKey("id"))
    {
        int id = Convert.ToInt32(context.Request.RouteValues["id"]);
        await context.Response.WriteAsync($"Products details - {id}");
    }
    else
    {
        await context.Response.WriteAsync($"Products details - id is not supplied");
    }
});
```

#### 정규 표현식 제약 조건 (Regular Expression Constraints)

`regex(pattern)` 제약 조건은 가장 강력하고 유연한 방법으로, 정규 표현식을 사용하여 거의 모든 종류의 복잡한 문자열 패턴을 정의할 수 있다.

**C# 코드 예시**

```csharp
// 연도(year)는 1900 이상의 정수여야 하고,
// 월(month)은 'apr', 'jul', 'oct', 'jan' 중 하나여야 한다.
endpoints.Map("sales-report/{year:int:min(1900)}/{month:regex(^(apr|jul|oct|jan)$)}", async context =>
{
    int year = Convert.ToInt32(context.Request.RouteValues["year"]);
    string? month = Convert.ToString(context.Request.RouteValues["month"]);

    await context.Response.WriteAsync($"sales report - {year} - {month}");
});
```

위 예제의 `regex(^(apr|jul|oct|jan)$)`는 다음을 의미한다.

- `^`: 문자열의 시작
- `(...)`: 괄호 안의 그룹
- `|`: OR 조건 (apr 또는 jul 또는 oct 또는 jan)
- `$`: 문자열의 끝

---

### 4. 핵심 고찰: 라우팅 제약 조건 vs 비즈니스 유효성 검사 🧐

라우팅 제약 조건은 매우 강력하지만, **모든 유효성 검사를 제약 조건으로 해결하려는 것은 좋지 않은 접근 방식**이다. Microsoft 공식 문서에서도 이 점을 강조한다.

- **라우팅 제약 조건의 역할**: 들어온 URL이 특정 엔드포인트에 **매칭되는지 여부를 결정**하는 것이다. 즉, '주소'가 올바른가? 에 대한 답이다. 조건이 맞지 않으면 404 Not Found를 반환하며, 클라이언트는 왜 요청이 실패했는지 구체적인 원인을 알기 어렵다.
- **비즈니스 유효성 검사의 역할**: 엔드포인트 로직 **내부에서** 매개변수 값이 비즈니스 규칙에 맞는지 검증하는 것이다. '주소는 맞는데, 내용물이 올바른가?' 에 대한 답이다. 검증에 실패하면 400 Bad Request 와 함께 "해당 월은 지원되지 않습니다." 와 같이 명확하고 친절한 오류 메시지를 반환할 수 있다.
    

**더 나은 접근 방식**

```csharp
// 제약 조건에서는 'month'가 문자열이라는 것만 확인한다.
endpoints.Map("sales-report/{year:int:min(1900)}/{month}", async context =>
{
    int year = Convert.ToInt32(context.Request.RouteValues["year"]);
    string? month = Convert.ToString(context.Request.RouteValues["month"]);

    // 실제 비즈니스 규칙은 핸들러 내부에서 검사한다.
    var allowedMonths = new List<string> { "apr", "jul", "oct", "jan" };
    if (allowedMonths.Contains(month))
    {
        await context.Response.WriteAsync($"sales report - {year} - {month}");
    }
    else
    {
        // 클라이언트에게 명확한 오류를 반환한다. (상태 코드 400 설정 로직 추가 가능)
        await context.Response.WriteAsync($"{month} is not allowed for sales report. Allowed months are: apr, jul, oct, jan.");
    }
});
```

**결론적으로, `int`, `guid` 와 같이 구조적으로 명확한 타입 검증은 제약 조건을 사용하고, 복잡한 비즈니스 규칙(예: 특정 값 목록, 복잡한 패턴)은 핸들러 내부에서 처리하여 더 의미 있는 응답을 제공하는 것이 바람직하다.**

---

### 5. Java Spring에서는? ☕️

Spring은 약간 다른 방식으로 비슷한 목표를 달성한다.

1. **자동 타입 변환 (Type Conversion)**: ASP.NET의 기본 타입 제약 조건(`int`, `guid` 등)은 Spring의 `@PathVariable` 타입 선언과 거의 동일하게 동작한다. URL의 값을 메서드 파라미터 타입(`Long`, `UUID` 등)으로 변환하려 시도하고, 실패하면 400 Bad Request를 응답한다.
    
    ```java
    @GetMapping("/cities/{cityId}")
    public String getCityInfo(@PathVariable UUID cityId) {
        // cityId가 유효한 UUID가 아니면 이 메서드는 실행되지 않는다.
        return "City information - " + cityId;
    }
    ```
    
2. **Bean Validation & 정규 표현식**: 길이, 범위, 정규식 같은 고급 제약 조건은 **Bean Validation API** 와 결합하여 더욱 선언적으로 처리한다. `@PathVariable`에 `@Size`, `@Min`, `@Max`, `@Pattern` 같은 어노테이션을 직접 붙여 사용한다.
    
    ```java
    // Spring Boot 프로젝트에 'spring-boot-starter-validation' 의존성 추가 필요
    import javax.validation.constraints.Pattern;
    import javax.validation.constraints.Size;
    import org.springframework.validation.annotation.Validated;
    
    @RestController
    @Validated // 클래스 레벨에 이 애너테이션을 붙여야 @PathVariable 유효성 검사가 활성화된다.
    public class EmployeeController {
    
        @GetMapping("/employee/profile/{employeeName}")
        public String getProfile(
            @Size(min = 4, max = 7) // 길이 제약
            @Pattern(regexp = "^[a-zA-Z]+$", message = "Employee name must be alphabetical") // 알파벳 제약
            @PathVariable String employeeName
        ) {
            return "In Employee profile - " + employeeName;
        }
    }
    ```
    

---

### 6. 전체 예제 코드

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// 라우팅 활성화
app.UseRouting();

// 엔드포인트 생성
app.UseEndpoints(endpoints =>
{
    // 예: employee/profile/john
    // 제약조건: 알파벳, 길이(4~7), 기본값 'harsha'
    endpoints.Map("employee/profile/{EmployeeName:alpha:length(4,7)=harsha}", async context =>
    {
        string? employeeName = Convert.ToString(context.Request.RouteValues["employeename"]);
        await context.Response.WriteAsync($"In Employee profile - {employeeName}");
    });


    // 예: products/details/500
    // 제약조건: 정수, 범위(1~1000), 선택적 매개변수
    endpoints.Map("products/details/{id:int:range(1,1000)?}", async context => {
        if (context.Request.RouteValues.ContainsKey("id"))
        {
            int id = Convert.ToInt32(context.Request.RouteValues["id"]);
            await context.Response.WriteAsync($"Products details - {id}");
        }
        else
        {
            await context.Response.WriteAsync($"Products details - id is not supplied");
        }
    });

    // 예: daily-digest-report/2025-07-30
    // 제약조건: datetime
    endpoints.Map("daily-digest-report/{reportdate:datetime}", async context =>
    {
        DateTime reportDate = Convert.ToDateTime(context.Request.RouteValues["reportdate"]);
        await context.Response.WriteAsync($"In daily-digest-report - {reportDate.ToShortDateString()}");
    });

    // 예: cities/ca761232-ed42-11ce-bacd-00aa0057b223
    // 제약조건: guid
    endpoints.Map("cities/{cityid:guid}", async context =>
    {
        Guid cityId = Guid.Parse(Convert.ToString(context.Request.RouteValues["cityid"])!);
        await context.Response.WriteAsync($"City information - {cityId}");
    });

    // 예: sales-report/2030/apr
    // 제약조건: 연도(1900 이상 정수), 월(regex 패턴)
    endpoints.Map("sales-report/{year:int:min(1900)}/{month:regex(^(apr|jul|oct|jan)$)}", async context =>
    {
        int year = Convert.ToInt32(context.Request.RouteValues["year"]);
        string? month = Convert.ToString(context.Request.RouteValues["month"]);
        await context.Response.WriteAsync($"sales report - {year} - {month}");
    });
});

// 일치하는 라우트가 없을 경우의 최종 응답
app.Run(async context => {
    await context.Response.WriteAsync($"No route matched at {context.Request.Path}");
});

app.Run();
```

### 결론

라우팅 제약 조건은 ASP.NET Core 애플리케이션의 안정성과 예측 가능성을 높이는 강력한 도구다. 하지만 그 역할을 명확히 이해하고 사용하는 것이 중요하다. **구조적인 타입 검증에는 제약 조건을 적극 활용**하되, **복잡한 비즈니스 규칙은 핸들러 내부에서 처리**하여 클라이언트에게 더 나은 피드백을 제공하는 균형 잡힌 접근 방식을 취하는 것이 좋다. 만약 내장 제약 조건으로 해결할 수 없는 매우 특수한 상황이 발생한다면, **사용자 지정 제약 조건(Custom Constraints)** 을 직접 만들어 사용할 수도 있다.