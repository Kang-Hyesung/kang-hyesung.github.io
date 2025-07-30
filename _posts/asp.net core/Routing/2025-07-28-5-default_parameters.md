---
title: "[5] 선택적 매개변수와 기본값 설정하기"
date: 2025-07-28 22:41 +0900
author: hyesung
---

웹 애플리케이션을 개발할 때, 우리는 종종 URL의 특정 부분을 동적으로 처리해야 한다. 예를 들어 `products/details/1`과 같은 URL에서 숫자 `1`은 특정 제품의 ID를 나타내는 **매개변수(Parameter)**다. ASP.NET Core의 강력한 **라우팅(Routing)** 시스템은 이런 동적 URL을 손쉽게 처리할 수 있도록 지원한다.

하지만 만약 사용자가 URL에 필수 매개변수 값을 입력하지 않는다면 어떻게 될까? 예를 들어, `products/details/`까지만 입력한다면 애플리케이션은 어떤 **엔드포인트(Endpoint)**에 요청을 보내야 할지 알 수 없어 오류를 반환하거나, 우리가 의도하지 않은 기본 페이지를 보여주게 된다.

이 글에서는 이러한 상황을 방지하고 더 유연하고 안정적인 API를 만들기 위해 ASP.NET Core에서 **선택적 매개변수(Optional Parameter)에 기본값을 설정하는 방법**을 알아본다.

### 기본 라우트 매개변수의 한계

먼저 기본값이 없는 라우트 매개변수의 동작을 살펴보자. 아래와 같이 파일 이름과 확장자를 받는 엔드포인트가 있다고 가정한다.

```csharp
// Eg: files/sample.txt
endpoints.Map("files/{filename}.{extension}", async context =>
{
    string? fileName = Convert.ToString(context.Request.RouteValues["filename"]);
    string? extension = Convert.ToString(context.Request.RouteValues["extension"]);
    await context.Response.WriteAsync($"In files - {fileName} - {extension}");
});
```

이 엔드포인트는 `files/hello.txt`와 같은 요청은 성공적으로 처리하지만, 만약 `files/hello`처럼 확장자(`extension`) 값을 제공하지 않으면 URL 패턴이 일치하지 않아 해당 엔드포인트가 실행되지 않는다. 결국 요청은 다른 곳으로 흘러가 `app.Run()`에 정의된 기본 응답을 반환하게 된다. 이는 우리가 기대하는 동작이 아니다.

### Route Parameter에 기본값 할당하기

이러한 문제를 해결하기 위해 ASP.NET Core는 라우팅을 정의할 때 매개변수에 직접 기본값을 할당하는 간단하고 직관적인 방법을 제공한다. C# 메서드에서 사용하는 기본 인수(Default Argument)와 매우 유사한 개념이다.

기본값 할당 문법:
{매개변수이름=기본값}

이 문법을 사용하면, URL에서 해당 매개변수 값이 생략되었을 때 지정된 기본값이 자동으로 사용된다.

#### 예제 1: 직원 프로필 조회 (문자열 기본값)

`employee/profile/john`처럼 직원 이름을 받아 프로필을 보여주는 엔드포인트를 생각해보자. 만약 직원 이름이 없다면, 기본값으로 'harsha'를 사용하도록 설정할 수 있다.

```csharp
// Eg: employee/profile/john
endpoints.Map("employee/profile/{EmployeeName=harsha}", async context =>
{
    string? employeeName = Convert.ToString(context.Request.RouteValues["employeename"]);
    await context.Response.WriteAsync($"In Employee profile - {employeeName}");
});
```

**동작 방식:**

- **`employee/profile/smith` 요청 시:** `employeeName` 값으로 "smith"가 사용된다.
    > 출력: `In Employee profile - smith`
    
- **`employee/profile` 요청 시:** `EmployeeName` 매개변수 값이 없으므로 기본값인 "harsha"가 사용된다.
    > 출력: `In Employee profile - harsha`
    

이처럼 사용자가 값을 제공하면 그 값이 우선적으로 사용되고, 제공하지 않을 때만 기본값이 적용되어 라우트가 성공적으로 매칭된다.

#### 예제 2: 제품 상세 정보 조회 (숫자 기본값)

더 실용적인 예시로, 제품 ID를 받아 상세 정보를 보여주는 엔드포인트를 만들어 보자. ID가 제공되지 않으면 기본적으로 ID가 `1`인 제품의 정보를 보여주도록 설정할 수 있다.

```csharp
// Eg: products/details/1
endpoints.Map("products/details/{id=1}", async context => {
    // RouteValues는 기본적으로 object 타입을 반환하므로, 적절한 타입으로 변환이 필요하다.
    int id = Convert.ToInt32(context.Request.RouteValues["id"]);
    await context.Response.WriteAsync($"Products details - {id}");
});
```

**핵심 포인트:** `context.Request.RouteValues`는 값을 `object` 타입으로 반환한다. 따라서 `id`를 정수형으로 사용하기 위해서는 `Convert.ToInt32()`와 같이 명시적인 형 변환이 반드시 필요하다.

**동작 방식:**

- **`products/details/30` 요청 시:** `id` 값으로 `30`이 사용된다.
    > 출력: `Products details - 30`
    
- **`products/details` 요청 시:** `id` 매개변수 값이 없으므로 기본값인 `1`이 사용된다.
    > 출력: `Products details - 1`
    

만약 여기서 기본값 설정 (`=1`)이 없었다면, `products/details` 요청은 이 엔드포인트와 매치되지 않았을 것이다. 이처럼 기본값 설정은 API의 유연성을 크게 향상시킨다.

### 전체 코드 예시 (C# ASP.NET Core)

아래는 위에서 설명한 모든 예제를 포함한 전체 `Program.cs` 파일이다.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// 라우팅 활성화
app.UseRouting();

// 엔드포인트 생성
app.UseEndpoints(endpoints =>
{
    // 예제 1: 기본값 없는 라우트
    // 요청 예: files/sample.txt
    endpoints.Map("files/{filename}.{extension}", async context =>
    {
        string? fileName = Convert.ToString(context.Request.RouteValues["filename"]);
        string? extension = Convert.ToString(context.Request.RouteValues["extension"]);
        await context.Response.WriteAsync($"In files - {fileName} - {extension}");
    });

    // 예제 2: 문자열 기본값 할당
    // 요청 예: employee/profile/john
    endpoints.Map("employee/profile/{EmployeeName=harsha}", async context =>
    {
        string? employeeName = Convert.ToString(context.Request.RouteValues["employeename"]);
        await context.Response.WriteAsync($"In Employee profile - {employeeName}");
    });

    // 예제 3: 숫자 기본값 할당
    // 요청 예: products/details/1
    endpoints.Map("products/details/{id=1}", async context =>
    {
        int id = Convert.ToInt32(context.Request.RouteValues["id"]);
        await context.Response.WriteAsync($"Products details - {id}");
    });
});

// 매칭되는 엔드포인트가 없을 경우 실행되는 폴백(Fallback) 미들웨어
app.Run(async context => {
    await context.Response.WriteAsync($"Request received at {context.Request.Path}");
});

app.Run();
```

---

## Java Spring에서는?

그렇다면 이러한 선택적 경로 변수(Path Variable)와 기본값 처리는 **Java Spring Boot** 환경에서 어떻게 구현할까? Spring Framework에서는 컨트롤러(Controller) 메서드의 애너테이션(Annotation)을 통해 매우 유사한 기능을 구현한다.

ASP.NET Core의 `endpoints.Map()`은 Spring Boot의 `@RequestMapping` 또는 `@GetMapping`과 같은 애너테이션과 유사한 역할을 한다. 매개변수 처리는 `@PathVariable` 애너테이션을 사용한다.

Spring에서 선택적 경로 변수를 처리하는 대표적인 방법은 `java.util.Optional`을 사용하거나 `required` 속성을 `false`로 설정하는 것이다.

#### `Optional<T>`를 사용한 처리

Java 8 이상에서 권장되는 방식으로, `Optional<T>`을 사용하여 경로 변수의 존재 여부를 명확하게 처리할 수 있다.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import java.util.Optional;

@RestController
public class ProductController {

    // ASP.NET Core의 "products/details/{id=1}" 와 유사한 기능
    @GetMapping(value = {"/products/details", "/products/details/{id}"})
    public String getProductDetails(@PathVariable(required = false) Optional<Integer> id) {
        // id 값이 존재하면 그 값을 사용하고, 없으면 기본값 1을 사용한다.
        int productId = id.orElse(1);
        return "Product details - " + productId;
    }

    // ASP.NET Core의 "employee/profile/{EmployeeName=harsha}" 와 유사한 기능
    @GetMapping(value = {"/employee/profile", "/employee/profile/{employeeName}"})
    public String getEmployeeProfile(@PathVariable(required = false) Optional<String> employeeName) {
        // employeeName 값이 존재하면 그 값을 사용하고, 없으면 기본값 "harsha"를 사용한다.
        String name = employeeName.orElse("harsha");
        return "In Employee profile - " + name;
    }
}
```

## 핵심 개념

- **ASP.NET Core `endpoints.Map(".../{id=1}")`**: 라우트 정의에서 기본값을 직접 설정한다.
- **Spring `@GetMapping` + `@PathVariable`**: `@GetMapping`의 `value` 속성에 경로 변수가 없는 URL(`"/products/details"`)과 있는 URL(`"/products/details/{id}"`)을 모두 등록한다. `@PathVariable`의 `required`를 `false`로 설정하고 `Optional<T>`로 파라미터를 받는다. 이후 코드 내에서 `orElse()` 메서드를 통해 기본값을 지정한다.
    

이처럼 프레임워크마다 문법과 구현 방식에는 차이가 있지만, **'URL 매개변수가 없을 경우를 대비해 기본값을 설정하여 API의 안정성과 유연성을 높인다'**는 핵심 목표는 동일하다.