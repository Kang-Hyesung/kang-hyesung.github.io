---
title: "[2] Controller 와 Routing"
date: 2025-08-05 22:12 +0900
author: hyesung
description: 설명
---
### 1. 컨트롤러(Controller)란 무엇인가?

컨트롤러는 MVC(Model-View-Controller) 아키텍처 패턴의 핵심 구성 요소로, 클라이언트의 요청(Request)을 받아들이는 첫 관문이다. 컨트롤러의 주요 책임은 다음과 같다.

- **요청 수신 및 분석**: HTTP 요청을 수신하고, 쿼리 스트링(Query String), 요청 본문(Request Body), 헤더(Header) 등에서 필요한 데이터를 읽는다.
- **입력값 검증 (Validation)**: 수신한 데이터가 비즈니스 규칙에 맞는지 유효성을 검사한다. 예를 들어, ID 값이 양수인지, 특정 형식에 맞는지 등을 확인한다.
- **비즈니스 로직 호출**: 데이터 처리를 위해 서비스나 모델 계층의 비즈니스 로직을 호출한다.
- **응답 생성 (Response Generation)**: 처리 결과를 바탕으로 클라이언트에게 반환할 응답(일반적으로 `IActionResult`)을 생성한다.

#### 컨트롤러 정의 규칙

ASP.NET Core에서 특정 클래스를 컨트롤러로 인식하게 하려면 다음 규칙 중 하나 이상을 따라야 한다.

1. **클래스 이름 접미사**: 클래스 이름 끝에 `Controller`를 붙인다. (예: `HomeController`, `ProductsController`)
2. **`[Controller]` 애트리뷰트**: 클래스 선언부 위에 `[Controller]` 애트리뷰트를 명시한다.

```csharp
// 규칙 1: 이름 끝에 'Controller' 접미사 사용 (가장 일반적인 방법)
public class HomeController : Controller
{
    // ...
}

// 규칙 2: [Controller] 애트리뷰트 사용
[Controller]
public class Home : Controller
{
    // ...
}
```

대부분의 실제 프로젝트에서는 가독성과 관례에 따라 첫 번째 방법, 즉 클래스 이름에 `Controller` 접미사를 붙이는 것을 선호한다. 또한, `Microsoft.AspNetCore.Mvc.Controller` 클래스를 상속받으면 `Content()`, `Ok()`, `NotFound()` 등 응답 생성을 돕는 여러 유용한 헬퍼 메서드(Helper Method)를 사용할 수 있다.

### 2. 애트리뷰트 라우팅 (Attribute Routing)

라우팅은 **"특정 URL 요청을 어떤 코드가 처리할지 결정하는 과정"** 이다. ASP.NET Core는 컨트롤러의 액션 메서드(Action Method) 위에 `[Route]` 애트리뷰트를 직접 명시하여 직관적으로 라우팅 규칙을 설정할 수 있도록 지원한다.

#### 기본 라우팅 및 다중 경로 설정

애플리케이션을 처음 실행했을 때, 기본 URL(예: `https://localhost:5001/`)로 접속하면 404 오류가 발생하는 것을 볼 수 있다. 이는 해당 경로와 매핑된 액션 메서드가 없기 때문이다. `[Route("/")]`를 추가하여 기본 URL 요청을 처리할 수 있다.

하나의 액션 메서드는 여러 개의 라우트 경로를 가질 수 있다.

```csharp
using Microsoft.AspNetCore.Mvc;

namespace ControllersExample.Controllers
{
    public class HomeController : Controller
    {
        // "/" 또는 "/home" 경로로 요청이 오면 Index() 메서드 실행
        [Route("/")]
        [Route("home")]
        public ContentResult Index()
        {
            // HTML 콘텐츠를 반환
            return Content("<h1>Welcome</h1> <h2>Hello from Index</h2>", "text/html");
        }

        [Route("about")]
        public string About()
        {
            return "Hello from About";
        }
    }
}
```

위 코드에서 `Index` 메서드는 웹사이트의 루트 URL(`"/"`)과 `"/home"` URL 두 경로에 모두 응답한다.

#### 라우트 매개변수와 제약 조건 (Parameters & Constraints)

라우팅은 URL의 특정 부분을 동적인 매개변수로 받을 수 있다. 이때 중괄호 `{}`를 사용한다. 또한, `:regex`와 같은 제약 조건을 추가하여 매개변수가 특정 형식을 따르도록 강제할 수 있다.

예를 들어, 10자리 숫자로 된 휴대폰 번호만 유효한 경로로 인정하고 싶다면 정규 표현식(Regular Expression)을 사용할 수 있다.

```csharp
using Microsoft.AspNetCore.Mvc;

namespace ControllersExample.Controllers
{
    public class HomeController : Controller
    {
        // ... (Index, About 메서드는 위와 동일) ...

        // "contact-us/01012345678" 와 같은 형식의 URL만 허용
        [Route("contact-us/{mobile:regex(^\\d{{10}}$)}")]
        public string Contact(string mobile) // URL의 {mobile} 부분이 매개변수로 전달됨
        {
            return $"Hello from Contact. Your mobile number is {mobile}";
        }
    }
}
```

여기서 URL 템플릿과 액션 메서드의 이름(`Contact`)은 서로 독립적이다. 즉, URL은 `"contact-us"`이지만, 이를 처리하는 메서드 이름은 `Contact`가 될 수 있다.

**정규 표현식 분석**: `^\\d{{10}}$`

- `^`: 문자열의 시작을 의미한다.
- `\\d`: 숫자 하나를 의미한다. C# 문자열에서 백슬래시(`\`) 자체를 표현하려면 이중 백슬래시(`\\`)를 사용해야 한다.
- `{{10}}`: 바로 앞의 패턴(`\\d`)이 10번 반복됨을 의미한다. 라우트 제약 조건에서 중괄호(`{}`)를 리터럴 문자로 사용하려면 이중 중괄호로 감싸야 한다.
- `$`: 문자열의 끝을 의미한다.
    

### 3. Java Spring에서는?

지금까지 살펴본 ASP.NET Core의 개념은 Java Spring 프레임워크에도 거의 동일하게 존재한다. C# 개발 경험이 있는 분이라면 쉽게 적응할 수 있다.

|ASP.NET Core (C#)|Java Spring|설명|
|---|---|---|
|`Controller` 클래스|`@RestController` 클래스|클라이언트의 HTTP 요청을 처리하는 컨트롤러 클래스를 정의한다.|
|`[Route]` 애트리뷰트|`@RequestMapping`, `@GetMapping` 등|URL 경로를 특정 컨트롤러 메서드와 매핑한다.|
|`IActionResult`, `ContentResult`|`ResponseEntity<T>`, `@ResponseBody`|서버의 응답을 표현한다.|
|`{id}` (라우트 매개변수)|`/{id}` (`@PathVariable`)|URL 경로의 일부를 변수로 사용한다.|
|`{mobile:regex(...)}`|`/{mobile:regex}` (`@PathVariable`)|경로 변수에 정규 표현식 제약 조건을 적용한다.|

#### Java Spring 코드 예시

위 C# 코드를 Java Spring으로 변환하면 다음과 같다.

```java
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HomeController {

    // "/" 또는 "/home" 경로로 요청이 오면 index() 메서드 실행
    @GetMapping(value = {"/", "/home"})
    public ResponseEntity<String> index() {
        // HTML 콘텐츠를 반환
        return ResponseEntity.ok()
                .contentType(MediaType.TEXT_HTML)
                .body("<h1>Welcome</h1> <h2>Hello from Index</h2>");
    }

    @GetMapping("/about")
    public String about() {
        return "Hello from About";
    }

    // "contact-us/01012345678" 와 같은 형식의 URL만 허용
    @GetMapping("/contact-us/{mobile:^\\d{10}$}")
    public String contact(@PathVariable String mobile) {
        return "Hello from Contact. Your mobile number is " + mobile;
    }
}
```

- `@RestController`: 이 클래스가 RESTful 웹 서비스의 컨트롤러임을 나타낸다. `@Controller`와 `@ResponseBody`가 합쳐진 형태다.
- `@GetMapping`: HTTP GET 요청을 처리하는 메서드에 사용되며, `value` 속성으로 여러 URL 경로를 지정할 수 있다.
- `ResponseEntity<String>`: HTTP 응답 상태 코드, 헤더, 본문을 모두 포함할 수 있는 객체다. C#의 `IActionResult`와 유사한 역할을 한다.
- `@PathVariable`: URL 경로에 포함된 변수(예: `{mobile}`)를 메서드 매개변수로 가져온다. 여기에 직접 정규 표현식을 작성하여 제약 조건을 설정할 수 있다.