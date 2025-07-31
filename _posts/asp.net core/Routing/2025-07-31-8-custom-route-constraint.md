---
title: "[8] Custom Route Constraint"
date: 2025-07-31 21:38 +0900
author: hyesung
---
API 엔드포인트를 설계할 때, 우리는 특정 형식을 따르는 경로 매개변수(Path Parameter)만 허용해야 하는 경우가 많다. 예를 들어, `sales-report/{year}/{month}`라는 경로에서 `month` 매개변수가 `apr`, `jul`, `oct`, `jan` 중 하나여야 한다는 규칙이 있다고 가정해 보자.

이러한 제약 조건을 애플리케이션의 여러 경로에서 반복적으로 사용해야 한다면, 정규 표현식(Regular Expression)을 매번 복사-붙여넣기 하는 것은 비효율적이며 유지보수를 어렵게 만든다.

ASP.NET Core에서는 이 문제를 해결하기 위해 **사용자 지정 경로 제약 조건(Custom Route Constraint)** 이라는 강력한 기능을 제공한다. 

## C# (ASP.NET Core)에서 사용자 지정 경로 제약 조건 구현하기

ASP.NET Core에서 사용자 지정 제약 조건은 특정 인터페이스를 구현하는 클래스로 정의된다. 이를 통해 복잡한 유효성 검사 로직을 캡슐화하고 재사용 가능한 구성 요소로 만들 수 있다.

### 1. 제약 조건 클래스 생성하기

먼저, 제약 조건 로직을 담을 클래스를 생성한다. 이 클래스는 반드시 `IRouteConstraint` 인터페이스를 구현해야 하며, `Match` 메서드를 포함해야 한다.

`Match` 메서드는 라우팅 시스템이 경로를 확인할 때 자동으로 호출되며, 반환 값( `true` 또는 `false` )에 따라 해당 경로의 일치 여부를 결정한다.

```csharp
// /CustomConstraints/MonthsCustomConstraint.cs

using System.Text.RegularExpressions;

namespace RoutingExample.CustomConstraints
{
    // IRouteConstraint 인터페이스를 구현한다.
    public class MonthsCustomConstraint : IRouteConstraint
    {
        // Match 메서드는 라우팅 시스템에 의해 자동으로 호출된다.
        public bool Match(HttpContext? httpContext, IRouter? route, string routeKey, RouteValueDictionary values, RouteDirection routeDirection)
        {
            // 1. 'values' 딕셔너리에 'routeKey' (예: "month")에 해당하는 값이 있는지 확인한다.
            if (!values.ContainsKey(routeKey))
            {
                return false; // 값이 없으면 일치하지 않음
            }

            // 2. 허용할 월(month)에 대한 정규 표현식을 정의한다.
            Regex regex = new Regex("^(apr|jul|oct|jan)$");
            string? monthValue = Convert.ToString(values[routeKey]);

            // 3. 정규 표현식을 사용하여 값의 유효성을 검사한다.
            if (regex.IsMatch(monthValue))
            {
                return true; // 값이 정규 표현식과 일치하면 true 반환
            }
            return false; // 일치하지 않으면 false 반환
        }
    }
}
```

### 2. 제약 조건 등록하기

생성한 제약 조건 클래스를 애플리케이션의 서비스 컨테이너에 등록해야 한다. `Program.cs` 파일에서 `AddRouting` 메서드를 사용하여 제약 조건을 특정 키(이 예제에서는 "months")와 매핑한다.

```csharp
// Program.cs

using RoutingExample.CustomConstraints;

var builder = WebApplication.CreateBuilder(args);

// 라우팅 서비스를 추가하고, 사용자 지정 제약 조건을 등록한다.
builder.Services.AddRouting(options => {
    // "months" 라는 키워드를 MonthsCustomConstraint 클래스와 매핑한다.
    options.ConstraintMap.Add("months", typeof(MonthsCustomConstraint));
});

var app = builder.Build();

// ... (이하 생략)
```

### 3. 경로에 제약 조건 적용하기

이제 등록된 제약 조건을 라우팅 규칙에 적용할 수 있다. `Map` 메서드에서 경로 매개변수 뒤에 콜론(`:`)과 함께 등록한 키("months")를 추가하면 된다.

```csharp
// Program.cs

app.UseEndpoints(endpoints =>
{
    // {month} 매개변수에 "months" 제약 조건을 적용한다.
    endpoints.Map("sales-report/{year:int:min(1900)}/{month:months}", async context =>
    {
        int year = Convert.ToInt32(context.Request.RouteValues["year"]);
        string? month = Convert.ToString(context.Request.RouteValues["month"]);

        await context.Response.WriteAsync($"sales report - {year} - {month}");
    });
});
```

이제 `/sales-report/2025/apr` 와 같은 요청은 성공적으로 이 엔드포인트에 도달하지만, `/sales-report/2025/feb` 와 같은 요청은 제약 조건에 의해 거부되고 다음 일치하는 경로를 찾거나 404 응답을 반환하게 된다.

---

## Java Spring에서는?

Java Spring 환경에서는 ASP.NET Core의 `IRouteConstraint`와 직접적으로 1:1 매칭되는 개념은 없지만, **Bean Validation** 을 활용한 **사용자 지정 유효성 검사 애노테이션(Custom Validation Annotation)** 을 통해 동일한 목표를 매우 우아하게 달성할 수 있다.

이는 컨트롤러(Controller) 메서드에 도달하기 전에 매개변수의 유효성을 검사하는 선언적인(declarative) 방식이다.

### 1. 의존성 추가하기

먼저 `pom.xml` (Maven) 또는 `build.gradle` (Gradle) 파일에 `spring-boot-starter-validation` 의존성을 추가하여 Bean Validation 기능을 활성화한다.

**Maven (`pom.xml`)**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 2. 사용자 지정 애노테이션 생성하기

재사용할 유효성 검사 규칙을 위한 애노테이션을 정의한다. 여기서는 `@ValidMonth` 라는 애노테이션을 생성한다.

```java
// /validation/ValidMonth.java

import jakarta.validation.Constraint;
import jakarta.validation.Payload;
import java.lang.annotation.*;

@Documented
@Constraint(validatedBy = MonthValidator.class) // 3단계에서 만들 검증 클래스를 지정
@Target({ ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidMonth {
    String message() default "Invalid month value. Allowed values are: apr, jul, oct, jan";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

### 3. 유효성 검증기(Validator) 클래스 생성하기

실제 유효성 검사 로직을 담고 있는 `ConstraintValidator` 를 구현한 클래스를 만든다. 이 클래스는 위에서 만든 `@ValidMonth` 애노테이션이 붙었을 때 동작한다.

```java
// /validation/MonthValidator.java

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;
import java.util.regex.Pattern;

public class MonthValidator implements ConstraintValidator<ValidMonth, String> {

    // Java Stream API와 Pattern.compile로 성능 최적화
    private static final Pattern MONTH_PATTERN = Pattern.compile("^(apr|jul|oct|jan)$");

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) {
            return false; // null 값은 유효하지 않음
        }
        // 정규 표현식과 일치하는지 확인
        return MONTH_PATTERN.matcher(value).matches();
    }
}
```

### 4. 컨트롤러에 애노테이션 적용하기

마지막으로, `@RestController` 클래스 레벨에 `@Validated` 애노테이션을 추가하고, 유효성을 검사할 `@PathVariable` 에 우리가 만든 `@ValidMonth` 애노테이션을 붙여준다.

```java
// /controller/SalesReportController.java

import com.yourpackage.validation.ValidMonth; // 패키지 경로는 실제 환경에 맞게 수정
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Validated // 클래스 레벨에서 유효성 검사를 활성화
public class SalesReportController {

    @GetMapping("/sales-report/{year}/{month}")
    public ResponseEntity<String> getSalesReport(
            @PathVariable int year,
            // @PathVariable에 사용자 지정 애노테이션을 적용한다.
            @PathVariable @ValidMonth String month) {

        // year에 대한 유효성 검사 (예: @Min(1900))도 추가할 수 있다.
        // 이 로직은 유효성 검사를 통과한 후에만 실행된다.
        String responseBody = String.format("Sales report - %d - %s", year, month);
        return ResponseEntity.ok(responseBody);
    }
}
```

유효하지 않은 `month` 값(예: `feb`)으로 요청을 보내면, Spring은 `ConstraintViolationException`을 발생시키고 기본적으로 **400 Bad Request** HTTP 상태 코드를 클라이언트에 응답한다. 이는 별도의 분기문 없이도 유효성 검사가 자동으로 처리됨을 의미한다.

## 결론

|기능|ASP.NET Core (C#)|Spring Boot (Java)|
|---|---|---|
|**핵심 개념**|`IRouteConstraint` 인터페이스 구현|`ConstraintValidator` 구현|
|**구현 방식**|1. 제약 조건 클래스 작성<br>2. 서비스에 등록<br>3. 경로에 키워드로 적용|1. 유효성 검사 애노테이션 정의<br>2. 검증기 클래스 작성<br>3. 컨트롤러 매개변수에 애노테이션 적용|
|**유사점**|복잡한 유효성 검사 로직을 캡슐화하고 재사용한다.||
|**차이점**|라우팅 시스템의 일부로 동작하며, 불일치 시 다음 경로를 탐색한다.|Bean Validation 표준의 일부로 동작하며, 불일치 시 예외를 발생시켜 400 에러를 응답한다.|

ASP.NET Core의 **사용자 지정 경로 제약 조건**과 Spring의 **사용자 지정 유효성 검사 애노테이션**은 서로 다른 방식으로 구현되지만, **'재사용 가능한 선언적 유효성 검사'** 라는 동일한 목표를 달성한다. 두 방식 모두 코드의 중복을 줄이고, 특정 도메인 규칙을 중앙에서 관리하여 애플리케이션의 유지보수성을 크게 향상시킨다.

하나의 경로에서만 사용되는 간단한 유효성 검사라면 엔드포인트 메서드 내에서 직접 처리하는 것이 더 나을 수 있다. 하지만 둘 이상의 위치에서 반복되는 규칙이라면, 이와 같은 패턴을 적용하는 것이 훨씬 더 깔끔하고 효율적인 해결책이 될 것이다.

