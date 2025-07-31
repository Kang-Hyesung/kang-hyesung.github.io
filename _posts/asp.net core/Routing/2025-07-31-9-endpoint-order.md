---
title: "[9] Endpoint 우선순위"
date: 2025-07-31 22:23 +0900
author: hyesung
---
웹 애플리케이션을 개발할 때, 동일한 URL 패턴에 여러 개의 엔드포인트(Endpoint)가 매칭되는 경우가 종종 발생한다. 예를 들어, `sales-report/{year}/{month}`와 `sales-report/2024/jan`이라는 두 개의 경로가 있을 때, 사용자가 `sales-report/2024/jan`으로 요청을 보내면 어떤 엔드포인트가 실행될까?

많은 개발자가 코드에 정의된 순서대로 처리될 것이라 생각하지만, ASP.NET Core의 라우팅 시스템은 **순서가 아닌 명확한 우선순위 규칙(Precedence Rules)** 에 따라 동작한다. 

## 🎯 ASP.NET Core의 라우팅 우선순위 4가지 규칙

ASP.NET Core는 URL이 여러 경로와 일치할 때, 더 "구체적인(specific)" 경로를 선택하기 위해 다음과 같은 4가지 핵심 규칙을 따른다.

### 1. 더 많은 세그먼트(Segment)를 가진 경로가 우선한다.

URL 경로는 '/'로 구분되는 세그먼트로 구성된다. 두 경로가 충돌할 경우, 더 많은 세그먼트를 가진 경로가 더 높은 우선순위를 갖는다.

- `/a/b/c` (3개 세그먼트) > `/a/b` (2개 세그먼트)

따라서 `a/b/c`로 들어온 요청은 두 경로 모두에 매칭될 수 있지만, 세그먼트가 3개인 첫 번째 경로가 선택된다.

### 2. 리터럴(Literal) 경로가 경로 매개변수(Route Parameter)보다 우선한다.

리터럴 경로는 변수 없이 고정된 텍스트로만 이루어진 경로다. 동일한 위치에 리터럴과 매개변수가 함께 있다면, 리터럴 경로가 더 구체적인 것으로 간주되어 우선 선택된다.

- `sales-report/2024/jan` > `sales-report/{year}/{month}`

만약 요청 URL이 `sales-report/2024/jan`이라면, 두 번째 경로에서는 `year`와 `month` 매개변수에 값을 할당할 수 있지만, 첫 번째 리터럴 경로와 완벽하게 일치하므로 첫 번째 경로가 실행된다. 하지만 요청이 `sales-report/2023/dec`와 같이 리터럴 경로와 일치하지 않으면, 매개변수를 사용하는 두 번째 경로가 선택된다.

### 3. 제약 조건(Constraint)이 있는 매개변수가 없는 매개변수보다 우선한다.

경로 매개변수에는 타입, 길이, 범위 등 다양한 제약 조건을 추가할 수 있다. 제약 조건이 있는 매개변수는 더 구체적인 의도를 가지므로 제약이 없는 매개변수보다 우선순위가 높다.

- `a/{id:int}` > `a/{id}`

`a/10`이라는 요청이 들어오면, `id`는 정수(int)이므로 제약 조건에 부합한다. 따라서 제약 조건이 있는 첫 번째 경로가 선택된다. 만약 `a/hello`처럼 정수가 아닌 값으로 요청이 온다면, 첫 번째 경로의 제약 조건에 맞지 않으므로 두 번째 경로가 처리하게 된다.

### 4. 일반 매개변수가 캐치-올(Catch-all) 매개변수보다 우선한다.

캐치-올 매개변수(`**` 또는 `*`)는 나머지 모든 URL 세그먼트를 포괄하는 특별한 매개변수다. 이는 가장 낮은 우선순위를 가지며, 일치하는 다른 경로가 없을 때 최후의 수단으로 사용된다.

- `a/{id}` > `a/{**catchall}`

`a/10` 요청은 두 경로 모두와 일치하지만, 더 구체적인 일반 매개변수를 가진 첫 번째 경로가 선택된다. 캐치-올 매개변수는 일반적으로 폴백(fallback) 라우팅이나 페이지 경로 처리 등에 유용하다.

---

## 💻 실전 예제로 이해하기

아래는 제공된 C# 코드를 바탕으로 라우팅 우선순위 규칙을 보여주는 예제다.

**`Program.cs`**

```csharp
var builder = WebApplication.CreateBuilder(args);

// ... 서비스 설정 ...

var app = builder.Build();

app.UseRouting();

app.UseEndpoints(endpoints =>
{
    // 규칙 1: 경로 매개변수를 사용하는 일반적인 판매 보고서 엔드포인트
    // 예: /sales-report/2023/dec
    endpoints.Map("sales-report/{year:int:min(1900)}/{month:alpha:length(3)}", async context =>
    {
        int year = Convert.ToInt32(context.Request.RouteValues["year"]);
        string? month = Convert.ToString(context.Request.RouteValues["month"]);
        await context.Response.WriteAsync($"Sales Report for: {year} - {month}");
    });

    // 규칙 2: 리터럴 텍스트를 사용하는 특정 판매 보고서 엔드포인트
    // 오직 /sales-report/2024/jan 요청에만 응답
    endpoints.Map("sales-report/2024/jan", async context =>
    {
        await context.Response.WriteAsync("Sales report exclusively for 2024 - jan");
    });
});

app.Run(async context => {
    await context.Response.WriteAsync($"No route matched at {context.Request.Path}");
});

app.Run();
```

위 코드에는 두 개의 엔드포인트가 정의되어 있다. 하나는 연도와 월을 매개변수로 받고, 다른 하나는 `2024/jan`이라는 고정된 리터럴 경로를 사용한다.

- 요청 URL: /sales-report/2024/jan
    
    이 URL은 두 엔드포인트 패턴에 모두 일치한다. 하지만 **규칙 2 (리터럴 경로 우선)**에 따라 endpoints.Map("sales-report/2024/jan", ...)가 선택된다. 따라서 응답은 "Sales report exclusively for 2024 - jan"이 된다.
    
- 요청 URL: /sales-report/2023/dec
    
    이 URL은 리터럴 경로와 일치하지 않는다. 하지만 매개변수를 사용하는 첫 번째 엔드포인트(sales-report/{year}/{month})의 패턴과는 일치한다. year에는 2023이, month에는 dec가 할당된다. 따라서 응답은 "Sales Report for: 2023 - dec"가 된다.
    

이 예제의 가장 큰 장점은 **개발자가 엔드포인트 정의 순서를 신경 쓸 필요가 없다는 점**이다. 리터럴 경로를 매개변수 경로보다 먼저 정의하든 나중에 정의하든, ASP.NET Core의 라우팅 시스템이 알아서 우선순위에 따라 올바른 엔드포인트를 실행해준다.

---

## 🤔 Java Spring에서는?

C#의 ASP.NET Core에서 경험한 라우팅 개념은 Java Spring Framework (특히 Spring MVC)에서도 매우 유사하게 적용된다. Spring에서는 컨트롤러(Controller)의 메서드에 `@RequestMapping` 또는 이의 단축형인 `@GetMapping`, `@PostMapping` 등의 애너테이션(Annotation)을 사용하여 URL을 매핑한다.

Spring 역시 **가장 구체적인 경로를 우선(most specific path wins)** 하는 원칙을 따른다.

#### C#과 Java Spring의 라우팅 매핑

|개념|ASP.NET Core (C#)|Spring (Java)|
|---|---|---|
|**경로 매핑**|`endpoints.Map(...)`|`@GetMapping(...)`|
|**경로 매개변수**|`/{year}`|`/{year}` + `@PathVariable`|
|**타입 제약**|`/{id:int}`|메서드 파라미터 타입을 `int`로 지정|
|**캐치-올**|`/{**path}`|`/**` (Ant-style path patterns)|

#### Spring 예제 코드

위의 C# 예제와 동일한 로직을 Java Spring으로 구현하면 다음과 같다.

**`SalesReportController.java`**

```java
@RestController
@RequestMapping("/sales-report")
public class SalesReportController {

    // 규칙 1: 경로 변수(PathVariable)를 사용하는 일반적인 판매 보고서 엔드포인트
    @GetMapping("/{year}/{month}")
    public String getSalesReport(@PathVariable int year, @PathVariable String month) {
        // 정규식 등으로 month 유효성 검증 추가 가능
        return String.format("Sales Report for: %d - %s", year, month);
    }

    // 규칙 2: 리터럴 경로를 사용하는 특정 판매 보고서 엔드포인트
    @GetMapping("/2024/jan")
    public String getExclusiveSalesReport() {
        return "Sales report exclusively for 2024 - jan";
    }
}
```

ASP.NET Core와 마찬가지로, `GET /sales-report/2024/jan` 요청이 들어오면 Spring은 더 구체적인 `/2024/jan` 경로가 매핑된 `getExclusiveSalesReport()` 메서드를 실행한다. 그 외의 `GET /sales-report/2023/dec`와 같은 요청은 `@PathVariable`을 사용하는 `getSalesReport()` 메서드에서 처리된다.

## 💡 결론

ASP.NET Core와 Java Spring의 라우팅 시스템은 모두 **정의 순서가 아닌 경로의 구체성(specificity)** 을 기반으로 동작한다.

1. **리터럴 경로**는 항상 **매개변수 경로**보다 우선한다.
2. **더 많은 세그먼트**나 **더 구체적인 제약 조건**을 가진 경로가 우선권을 갖는다.
3. 이러한 규칙 덕분에 개발자는 엔드포인트의 물리적 코드 순서에 대해 걱정할 필요 없이, 논리적으로 예측 가능한 애플리케이션을 구축할 수 있다.