---
title: "[10] UseStaticFiles()"
date: 2025-08-01 21:41 +0900
author: hyesung
---
웹 애플리케이션을 개발할 때 이미지, CSS, JavaScript 파일과 같은 정적(Static) 콘텐츠를 다루는 것은 필수적이다. ASP.NET Core는 이러한 정적 파일을 제공하기 위한 강력하고 유연한 메커니즘을 제공한다.

### 1. 정적 파일이란 무엇인가?

정적 파일은 서버에서 별도의 처리 없이 클라이언트(브라우저)에게 그대로 전달되는 파일을 의미한다. 대표적인 예는 다음과 같다.

- 이미지 파일 (`.jpg`, `.png`, `.gif`)
- 스타일시트 파일 (`.css`)
- 자바스크립트 파일 (`.js`)
- 텍스트 및 문서 파일 (`.txt`, `.pdf`)

ASP.NET Core는 보안상의 이유로 기본적으로 프로젝트 내의 모든 파일에 대한 직접적인 접근을 허용하지 않는다. 따라서 정적 파일을 클라이언트가 접근할 수 있도록 하려면 명시적인 설정이 필요하다.

---

### 2. 기본 설정: `UseStaticFiles`와 `wwwroot` 폴더

ASP.NET Core에서 정적 파일을 제공하는 가장 기본적인 방법은 `UseStaticFiles` 미들웨어(Middleware)를 사용하는 것이다. 이 미들웨어는 특정 폴더의 파일을 웹을 통해 접근할 수 있도록 허용한다.
ASP.NET Core의 기본 규약(Convention)에 따르면, 모든 정적 파일은 **`wwwroot`** 라는 이름의 폴더에 위치시키는 것을 권장한다.

#### 단계별 구현

1. **`wwwroot` 폴더 생성**: 프로젝트 루트에 `wwwroot`라는 이름의 폴더를 생성한다. Visual Studio에서는 이 폴더를 생성하면 자동으로 웹 콘텐츠 루트임을 나타내는 특수한 아이콘이 표시된다.
2. **정적 파일 추가**: 생성한 `wwwroot` 폴더 안에 이미지, PDF 등 원하는 정적 파일을 추가한다.
3. **미들웨어 등록 (`Program.cs`)**: `Program.cs` 파일에서 `app.UseStaticFiles()`를 호출하여 미들웨어를 파이프라인에 추가한다.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// `wwwroot` 폴더의 정적 파일을 제공하도록 설정한다.
// 이 코드는 라우팅(UseRouting) 미들웨어보다 앞에 위치하는 것이 좋다.
app.UseStaticFiles(); 

app.MapGet("/", () => "Hello World!");

app.Run();
```

이제 애플리케이션을 실행하고 브라우저에서 `https://<your-domain>/<file-name>` 형식으로 요청하면 `wwwroot` 폴더 안의 해당 파일이 정상적으로 제공된다. 예를 들어 `wwwroot/img1.jpg` 파일은 `https://<your-domain>/img1.jpg`로 접근할 수 있다.

> **보안 강화**: 이 방식은 `wwwroot` 폴더 외부의 파일(예: `Program.cs`나 다른 소스 코드 파일)이 웹을 통해 절대 노출되지 않도록 보장하여 애플리케이션의 보안을 크게 향상시킨다.

---

### 3. 고급 설정 1: 웹 루트 폴더 이름 변경하기

`wwwroot`는 규약일 뿐, 강제는 아니다. 만약 다른 이름의 폴더를 웹 루트로 사용하고 싶다면 `WebApplicationOptions`를 통해 간단히 변경할 수 있다.

예를 들어, 웹 루트 폴더를 `myroot`로 변경해보자.

1. **폴더 이름 변경**: `wwwroot` 폴더의 이름을 `myroot`로 변경한다.
2. **`WebApplicationOptions` 설정 (`Program.cs`)**: `WebApplication.CreateBuilder`를 호출할 때 `WebApplicationOptions` 객체를 전달하여 `WebRootPath`를 지정한다.
    

```csharp
var builder = WebApplication.CreateBuilder(new WebApplicationOptions()
{
    // 기본 웹 루트 경로를 'myroot'로 지정한다.
    WebRootPath = "myroot"
});

var app = builder.Build();

// 이제 이 UseStaticFiles()는 'myroot' 폴더를 기준으로 작동한다.
app.UseStaticFiles();

app.MapGet("/", () => "Hello World!");

app.Run();
```

이제부터 `UseStaticFiles()` 미들웨어는 `wwwroot`가 아닌 `myroot` 폴더를 기본 웹 루트로 인식하고 파일을 제공한다.

---

### 4. 고급 설정 2: 여러 개의 정적 파일 폴더 사용하기

프로젝트 구조에 따라 여러 위치에 있는 정적 파일을 제공해야 할 수도 있다. 예를 들어, 기본 웹 루트인 `myroot` 외에 `mywebroot`라는 폴더의 파일도 함께 제공하고 싶다고 가정해보자.
이 경우, `UseStaticFiles` 미들웨어를 추가로 호출하고 `StaticFileOptions`를 통해 경로를 직접 지정해주면 된다.

#### 최종 코드 예시

```csharp
// Microsoft.Extensions.FileProviders 네임스페이스를 사용해야 한다.
using Microsoft.Extensions.FileProviders;

var builder = WebApplication.CreateBuilder(new WebApplicationOptions()
{
    WebRootPath = "myroot"
});

var app = builder.Build();

// 1. 기본 웹 루트('myroot')에 대한 정적 파일 제공을 활성화한다.
app.UseStaticFiles(); 

// 2. 'mywebroot' 폴더에 대한 정적 파일 제공을 추가로 활성화한다.
app.UseStaticFiles(new StaticFileOptions()
{
    // 파일 제공자의 루트를 물리적인 경로로 지정한다.
    FileProvider = new PhysicalFileProvider(
        // Path.Combine을 사용하여 안전하게 경로를 조합한다.
        // builder.Environment.ContentRootPath는 프로젝트의 최상위 경로를 의미한다.
        Path.Combine(builder.Environment.ContentRootPath, "mywebroot")
    )
});

app.UseRouting();

app.UseEndpoints(endpoints =>
{
    endpoints.Map("/", async context =>
    {
        await context.Response.WriteAsync("Hello from root!");
    });
});

app.Run();
```

**코드 설명:**

- 첫 번째 `app.UseStaticFiles()` 호출은 `WebRootPath`로 설정된 `myroot` 폴더를 대상으로 한다.
- 두 번째 `app.UseStaticFiles()` 호출은 `StaticFileOptions`를 인자로 받는다.
    
    - **`FileProvider`**: 정적 파일의 위치를 알려주는 역할을 한다.
    - **`PhysicalFileProvider`**: 파일 시스템의 특정 물리적 경로를 가리킨다.
    - **`Path.Combine(...)`**: 운영체제에 맞는 경로 구분자를 사용하여 안전하게 절대 경로를 생성한다. `builder.Environment.ContentRootPath`는 애플리케이션이 실행되는 루트 디렉터리 경로다.

이제 애플리케이션은 `myroot` 폴더와 `mywebroot` 폴더 양쪽에 있는 정적 파일을 모두 제공할 수 있다. 요청이 들어오면 미들웨어 호출 순서에 따라 `myroot`에서 먼저 파일을 찾고, 없으면 `mywebroot`에서 찾아 제공한다.

---

### Java Spring에서는?

C#의 `ASP.NET Core`에서 정적 파일을 다루는 방식은 Java의 `Spring Boot`와 유사점과 차이점을 가진다.

- **기본 정적 리소스 경로 (↔ `wwwroot`)**: Spring Boot는 별도의 설정 없이 기본적으로 특정 경로에 있는 정적 리소스를 자동으로 제공한다. 이 경로는 `classpath` 기준으로 다음과 같다.
    
    - `/static`
    - `/public`
    - `/resources`
    - `/META-INF/resources`
      wwwroot 폴더를 만드는 것처럼, `src/main/resources/static` 폴더에 이미지를 넣으면 애플리케이션 실행 시 바로 접근할 수 있다. `UseStaticFiles()` 같은 명시적인 미들웨어 선언이 필요 없는 것이 가장 큰 차이점이다.
        
- **정적 리소스 경로 변경 (↔ `WebRootPath`)**: Spring Boot에서 정적 리소스 경로를 변경하거나 추가하고 싶을 때는 `application.properties` 또는 `application.yml` 파일에서 다음 속성을 설정한다.
    
    **`application.properties` 예시:**
    
  ```
    # 기본 경로 대신 classpath:/my-static/ 경로를 사용하도록 설정
    spring.web.resources.static-locations=classpath:/my-static/
    
    # 여러 경로를 지정할 수도 있다.
    # spring.web.resources.static-locations=classpath:/my-static/,classpath:/another-static/
    ```
    
    이 방식은 `WebApplicationOptions`를 통해 웹 루트를 변경하는 것과 개념적으로 동일하다.

---

### 결론

ASP.NET Core에서 정적 파일을 제공하는 것은 `UseStaticFiles` 미들웨어를 통해 간단히 시작할 수 있다. 기본적으로 `wwwroot` 폴더 규칙을 따르는 것이 편리하지만, 프로젝트의 요구사항에 따라 `WebRootPath`를 변경하거나 `StaticFileOptions`를 사용하여 여러 폴더를 유연하게 구성할 수 있다. 이러한 기능은 애플리케이션의 구조를 깔끔하게 유지하고 보안을 강화하는 데 핵심적인 역할을 한다.
