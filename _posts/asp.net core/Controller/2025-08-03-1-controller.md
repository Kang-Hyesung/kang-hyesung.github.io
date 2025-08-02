---
title: "[1] Controller 생성"
date: 2025-08-03 00:57 +0900
author: hyesung
---
### 왜 컨트롤러가 필요한가?

초기 웹 애플리케이션을 개발할 때, 우리는 종종 `Program.cs` 파일 하나에 모든 URL 엔드포인트(Endpoint)와 비즈니스 로직을 작성하는 실수를 범한다. 간단한 프로젝트에서는 이 방식이 빠르고 편리하게 느껴질 수 있다.

하지만 실제 서비스를 운영하는 대규모 프로젝트를 상상해보자. 수백, 수천 개의 엔드포인트와 그에 따른 방대한 양의 코드가 단 하나의 파일에 뒤섞여 있다면 어떨까? 이는 마치 잘 정리된 도서관이 아닌, 모든 책이 바닥에 널브러져 있는 창고와 같다. 코드의 가독성은 현저히 떨어지고, 기능 추가나 오류 수정(Debugging)은 재앙에 가까워진다.

이러한 문제를 해결하기 위해 등장한 개념이 바로 **컨트롤러(Controller)** 다. 컨트롤러는 관련된 기능들을 하나의 논리적인 단위로 묶어주는 역할을 한다.

### 컨트롤러(Controller)란 무엇인가?

**컨트롤러**는 **액션 메서드(Action Method)** 들의 집합을 포함하는 C# 클래스다. 여기서 각 액션 메서드는 특정 URL 요청을 처리하는 하나의 **엔드포인트**가 된다.

예를 들어, 사용자 관리 기능을 생각해보자.

- `Action 1`: 사용자 회원가입 처리
- `Action 2`: 사용자 로그인 처리
- `Action 3`: 사용자 정보 수정 처리

이 세 가지 액션은 모두 '사용자 계정 관리'라는 공통된 논리적 목적을 가진다. 따라서 이들을 `UserAccountController`라는 하나의 컨트롤러 클래스로 그룹화하여 관리하는 것이 바람직하다. 이처럼 컨트롤러는 관련 있는 액션들을 묶어 코드의 구조를 체계적으로 만들고, 유지보수성을 극적으로 향상시킨다.

### ASP.NET Core에서 컨트롤러 생성 및 활성화하기

이제 실제로 ASP.NET Core 프로젝트에서 컨트롤러를 어떻게 만들고 활성화하는지 단계별로 알아보자.

#### 1단계: 컨트롤러 클래스 작성

가장 먼저, 컨트롤러 클래스를 담을 폴더를 생성한다. 규칙에 따라 프로젝트 최상단에 `Controllers` 라는 이름의 폴더를 만드는 것이 일반적이다.

그 다음, `Controllers` 폴더 안에 다음과 같이 `HomeController.cs` 파일을 작성한다.

```csharp
// Controllers/HomeController.cs
using Microsoft.AspNetCore.Mvc;

namespace ControllersExample.Controllers
{
  // 1. [Controller] 애트리뷰트를 클래스에 적용한다.
  [Controller]
  public class HomeController
  {
    // 2. 각 메서드에 [Route] 애트리뷰트로 URL 경로를 지정한다.
    [Route("/")]        // 루트 URL (http://localhost:port/)
    [Route("home")]     // /home URL (http://localhost:port/home)
    public string Index()
    {
      return "Hello from Index";
    }

    [Route("about")]    // /about URL
    public string About()
    {
      return "Hello from About";
    }

    // 3. 정규식을 사용한 라우트 제약 조건 추가
    [Route("contact-us/{mobile:regex(^\\d{{10}}$)}")]
    public string Contact()
    {
      return "Hello from Contact";
    }
  }
}
```

컨트롤러를 정의할 때 기억해야 할 핵심 규칙은 다음과 같다.

1. **클래스 이름**: 클래스의 이름은 반드시 `Controller` 접미사로 끝나야 한다. (예: `HomeController`, `ProductController`) ASP.NET Core는 이 규칙을 통해 해당 클래스를 컨트롤러로 인식한다.
2. **`[Controller]` 애트리뷰트**: 클래스 위에 `[Controller]` 애트리뷰트를 명시하여 이 클래스가 API 컨트롤러의 역할을 수행함을 프레임워크에 알린다.
3. **액션 메서드와 라우팅**: 클래스 내의 `public` 메서드들이 액션 메서드가 된다. 각 메서드 위에는 `[Route("url-path")]`와 같은 **애트리뷰트 라우팅(Attribute Routing)** 방식을 사용하여 해당 액션을 호출할 URL 경로를 직접 지정한다.

> 💡 **참고**: 코드 예시의 `[Route("contact-us/{mobile:regex(^\\d{{10}}$)}")]`는 URL 경로에 10자리 숫자만 허용하는 **라우트 제약 조건(Route Constraint)** 을 적용한 예시다. 이를 통해 유효하지 않은 형식의 요청을 컨트롤러 로직이 실행되기 전에 차단할 수 있다.

#### 2단계: 컨트롤러 서비스 등록 및 라우팅 활성화

컨트롤러 클래스를 작성했다고 해서 즉시 웹 요청을 처리할 수 있는 것은 아니다. ASP.NET Core 애플리케이션이 컨트롤러의 존재를 인지하고, 들어온 요청을 올바른 액션 메서드로 연결해주도록 설정해야 한다. 이 설정은 `Program.cs` 파일에서 이루어진다.

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// 1. 컨트롤러 서비스를 의존성 주입(DI) 컨테이너에 등록한다.
// 이 메서드는 프로젝트 내의 모든 컨트롤러 클래스들을 찾아 서비스로 추가해준다.
builder.Services.AddControllers();

var app = builder.Build();

// 2. 컨트롤러에 정의된 라우트 정보를 기반으로 엔드포인트를 매핑한다.
// 이 메서드가 있어야 [Route] 애트리뷰트가 동작한다.
app.MapControllers();

app.Run();
```

여기서 두 가지 핵심 메서드가 사용된다.

1. `builder.Services.AddControllers()`: 이 코드는 **의존성 주입(Dependency Injection, DI)** 시스템에 모든 컨트롤러를 서비스로 등록한다. 이렇게 등록되어야 런타임에 ASP.NET Core가 요청에 맞는 컨트롤러의 인스턴스를 생성하고 관리할 수 있다. 수십 개의 컨트롤러가 있더라도 이 한 줄이면 모두 자동으로 등록된다.
2. `app.MapControllers()`: 이 코드는 애플리케이션의 라우팅 시스템을 활성화하고, 각 컨트롤러의 액션 메서드에 정의된 `[Route]` 애트리뷰트를 읽어 실제 엔드포인트로 매핑하는 역할을 한다. 이 코드가 없다면 브라우저가 `/home`이나 `/about`으로 요청을 보내도 해당 액션 메서드를 찾지 못해 404 오류가 발생한다.

이제 애플리케이션을 실행하고 웹 브라우저에서 `http://localhost:[포트번호]/`, `http://localhost:[포트번호]/home`, `http://localhost:[포트번호]/about` 등의 주소로 접속하면 각각의 액션 메서드가 반환하는 문자열을 확인할 수 있다.

---

### Java Spring에서는?

그렇다면 이러한 컨트롤러 개념은 Java Spring 생태계에서 어떻게 구현될까? 놀랍도록 유사한 구조를 가지고 있다.

|C# ASP.NET Core|Java Spring Boot|설명|
|---|---|---|
|`[Controller]` 또는 `[ApiController]`|`@RestController`|해당 클래스가 웹 요청을 처리하는 컨트롤러임을 나타내는 애너테이션(Annotation)이다.|
|`[Route("...")]`, `[HttpGet]`, `[HttpPost]`|`@RequestMapping`, `@GetMapping`, `@PostMapping`|URL 경로와 HTTP 메서드(GET, POST 등)를 특정 메서드에 매핑하는 역할을 한다.|
|`builder.Services.AddControllers()`|**Auto-Configuration (자동 구성)**|Spring Boot는 클래스패스에 `@RestController`가 존재하면 자동으로 웹 애플리케이션으로 동작하도록 관련 빈(Bean)들을 등록한다. 별도의 명시적인 등록 코드가 필요 없다.|
|`app.MapControllers()`|**Auto-Configuration (자동 구성)**|`@RestController`와 `@RequestMapping` 계열 애너테이션을 스캔하여 자동으로 엔드포인트를 매핑한다. 이 또한 Spring Boot의 자동 구성 기능 덕분에 개발자가 신경 쓸 필요가 없다.|

다음은 위 C# 예제와 동일한 기능을 하는 Java Spring Boot 컨트롤러 코드다.

```java
// src/main/java/com/example/demo/HomeController.java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.PathVariable;


@RestController // @Controller + @ResponseBody, RESTful API 컨트롤러임을 명시
public class HomeController {

    @GetMapping({"/", "/home"}) // GET 요청에 대해 루트(/) 경로와 /home 경로를 매핑
    public String index() {
        return "Hello from Index";
    }

    @GetMapping("/about") // GET 요청에 대해 /about 경로를 매핑
    public String about() {
        return "Hello from About";
    }

    // Spring에서는 정규식을 사용하여 경로 변수를 검증한다.
    @GetMapping("/contact-us/{mobile:^\\d{10}$}")
    public String contact(@PathVariable String mobile) {
        return "Hello from Contact, mobile: " + mobile;
    }
}
```

보시다시피, 애트리뷰트(C#)가 애너테이션(Java)으로 바뀌었을 뿐, 클래스와 메서드 단위로 라우팅을 정의하고 기능을 그룹화하는 핵심 철학은 완전히 동일하다. 특히 Spring Boot의 **자동 구성(Auto-Configuration)** 덕분에 `Program.cs`에서 했던 명시적인 서비스 등록 및 매핑 과정이 필요 없다는 점이 큰 차이점이다.

---

### 결론

컨트롤러는 단순히 코드를 분리하는 것을 넘어, 애플리케이션을 논리적이고 기능적인 단위로 구조화하는 핵심적인 디자인 패턴이다. 컨트롤러를 올바르게 사용하면 대규모 프로젝트에서도 각 기능의 책임과 역할이 명확해져 유지보수가 용이하고 확장성 높은 애플리케이션을 구축할 수 있다.