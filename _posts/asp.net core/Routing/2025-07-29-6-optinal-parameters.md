---
title: "[6] 선택적 매개변수 - null"
date: 2025-07-29 22:11 +0900
author: hyesung
---
API 엔드포인트(Endpoint)를 설계할 때, URL 경로에 포함된 특정 값(매개변수)이 항상 제공되지 않을 수도 있다. 예를 들어, `products/details/101`처럼 ID를 포함할 수도 있지만, `products/details`처럼 ID 없이 요청하는 경우도 고려해야 한다.

ASP.NET Core 라우팅 시스템은 이러한 시나리오를 '선택적 매개변수(Optional Parameter)'라는 강력하고 직관적인 기능으로 해결한다. 

## 🤔 문제 상황: 매개변수가 제공되지 않을 경우

만약 `products/details/{id}`와 같이 라우트를 정의했다고 가정해 보자. 이 경우, `{id}`는 필수 매개변수가 된다. 사용자가 `products/details/`까지만 요청하고 ID 값을 생략하면, 이 요청은 해당 엔드포인트와 매칭되지 않아 404 Not Found 오류를 반환하게 된다.

하지만 요구사항에 따라 ID가 없는 요청도 정상적으로 처리하고 싶을 때가 있다. "ID가 제공되지 않았습니다"와 같은 안내 메시지를 보여주거나, 전체 상품 목록을 보여주는 등의 다른 로직을 수행해야 할 수 있다.

## ✅ 해결책: C#의 선택적 라우트 매개변수 ?

ASP.NET Core에서는 라우트 매개변수 이름 뒤에 물음표(`?`)를 붙이는 것만으로 간단하게 이 문제를 해결할 수 있다.

```csharp
//Eg: products/details/ 또는 products/details/101
endpoints.Map("products/details/{id?}", async context => {
    // 'id' 키가 라우트 값에 포함되어 있는지 먼저 확인한다.
    if (context.Request.RouteValues.ContainsKey("id"))
    {
        // 키가 존재하면 값을 가져와 정수로 변환한다.
        int id = Convert.ToInt32(context.Request.RouteValues["id"]);
        await context.Response.WriteAsync($"Products details - {id}");
    }
    else
    {
        // 키가 존재하지 않으면(URL에 id가 없으면) 대체 응답을 보낸다.
        await context.Response.WriteAsync($"Products details - id is not supplied");
    }
});
```

### 코드 분석

1. **`{id?}`**: 라우트 패턴에서 `id` 매개변수 뒤에 `?`를 추가했다. 이것이 바로 `id`가 **선택적**이라는 것을 프레임워크에 알리는 신호다. 이제 `products/details`와 `products/details/101` 두 가지 형식의 URL 모두 이 엔드포인트와 매칭된다.
2. **`null` 처리**: 사용자가 `id` 값을 제공하지 않으면, `context.Request.RouteValues["id"]`의 값은 `null`이 된다.
    
    > ⚠️ **주의**: `Convert.ToInt32(null)`을 호출하면 예외가 발생하는 대신 `0`을 반환한다. 이는 `Convert.ToInt32`의 기본 동작 방식이다. 따라서 `id`가 실제로 `0`으로 들어온 것인지, 아니면 `null`이라서 `0`으로 변환된 것인지 구분할 수 없다.
    
3. **`ContainsKey("id")`**: 이러한 모호성을 피하기 위해, 값에 직접 접근하기 전에 `context.Request.RouteValues.ContainsKey("id")`를 사용해 `id`라는 키 자체가 존재하는지 확인하는 것이 가장 안전하고 명확한 방법이다. 키가 존재할 때만 값을 읽어오고, 그렇지 않다면 값이 제공되지 않았다고 판단하여 분기 처리를 할 수 있다.

이처럼 선택적 매개변수를 활용하면, 값이 제공되지 않았을 때 데이터베이스 조회를 건너뛰는 등 불필요한 작업을 막고 더 유연하고 안정적인 API를 만들 수 있다.

---

## ☕ Java Spring에서는? @PathVariable과 Optional

그렇다면 Java Spring 환경에서는 이러한 선택적 경로 변수를 어떻게 다룰까? Spring MVC에서는 `@PathVariable` 어노테이션을 사용하여 URL의 일부를 매개변수로 받아온다.
핵심은 Java 8에서 도입된 **`Optional<T>`** 클래스를 함께 사용하는 것이다. 이는 '값이 없을 수도 있는' 변수를 명확하게 표현하는 방식이다.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import java.util.Optional;

@RestController
public class ProductController {

    // GET /products/details 또는 /products/details/101
    @GetMapping(value = {"/products/details", "/products/details/{id}"})
    public String getProductDetails(@PathVariable(required = false) Optional<Integer> id) {
        // Optional 객체의 isPresent() 메서드로 값이 존재하는지 확인한다.
        if (id.isPresent()) {
            // 값이 존재하면 get() 메서드로 실제 값을 얻는다.
            return "Products details - " + id.get();
        } else {
            // 값이 존재하지 않으면 대체 응답을 반환한다.
            return "Products details - id is not supplied";
        }
    }
}
```

### 코드 분석 및 비교

|ASP.NET Core (.NET)|Java Spring|설명|
|---|---|---|
|`endpoints.Map(".../{id?}")`|`@GetMapping({... , "/{id}"})`|C#은 `?`로 선택성을 표시하고, Spring은 여러 URL 패턴을 배열로 지정할 수 있다.|
|`매개변수 타입`|`@PathVariable(required=false)`|Spring에서는 `@PathVariable`의 `required` 속성을 `false`로 설정하여 선택적임을 명시한다.|
|`Nullable<T>` 또는 `T?`|`Optional<T>`|**(핵심 비교)** .NET의 Nullable 타입(`int?`)과 유사하게, Spring에서는 `Optional<T>`을 사용해 값의 존재 여부를 안전하게 다룬다.|
|`RouteValues.ContainsKey("id")`|`id.isPresent()`|값이 실제로 존재하는지 확인하는 로직이다. `Optional`을 사용하면 `NullPointerException` 걱정 없이 안전하게 검사할 수 있다.|

Spring의 `Optional`은 명시적으로 "값이 없을 수 있음"을 코드 수준에서 강제하므로, null 체크를 잊어버리는 실수를 방지하는 데 큰 도움이 된다.

## ✨ 결론

라우트 매개변수의 **선택적 처리**는 유연하고 사용자 친화적인 API를 만들기 위한 핵심 기술이다.

- **ASP.NET Core**에서는 라우트 패턴에 **`?`** 접미사를 붙이는 직관적인 방법으로 선택적 매개변수를 구현한다.
- **Java Spring**에서는 `@PathVariable(required = false)`와 **`Optional<T>`**을 조합하여 타입 안정성을 높이는 방식으로 동일한 목표를 달성한다.
