---
title: "[4] Route Parameters"
date: 2025-07-28 21:45 +0900
author: hyesung
---


### ASP.NET Core에서 동적 URL 만들기

웹 애플리케이션을 만들다 보면 고정된 주소만으로는 부족할 때가 많다. 예를 들어,

- `files/report.pdf` 요청이 오면 **report.pdf** 파일을 보여주고, `files/image.png` 요청이 오면 **image.png** 파일을 보여줘야 한다.
- `employee/profile/john` 이라고 요청하면 **john**의 프로필을, `employee/profile/jane` 이라고 하면 **jane**의 프로필을 보여줘야 한다.
    

여기서 공통점을 발견할 수 있다. URL의 앞부분(`files/`나 `employee/profile/`)은 고정되어 있고, 뒷부분(`report.pdf`, `image.png`, `john`, `jane`)은 계속 바뀌고 있다.

- **리터럴 텍스트 (Literal Text):** `files/`처럼 고정된 부분이다. 말 그대로 "문자 그대로의" 텍스트다.
- **라우트 매개변수 (Route Parameter):** `report.pdf`나 `john`처럼 계속 바뀌는 부분이다. 이 자리에 어떤 값이든 올 수 있다는 걸 알려주는 '자리표시자(Placeholder)' 같은 역할이다.
    

ASP.NET Core에서는 이 라우트 매개변수를 중괄호 `{}`를 사용해서 표현한다.

- `files/{filename}.{extension}`
- `employee/profile/{employeeName}`

이렇게 정의하면 `{}` 안의 이름으로 URL에 들어온 동적인 값을 코드 안에서 꺼내 쓸 수 있게 된다. 멋지다. 😉

---

### 💻 코드로 확인(C# / ASP.NET Core)

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// 라우팅 기능을 활성화한다. "이제부터 주소 매칭할 준비를 하자!"는 뜻이다.
app.UseRouting();

// UseEndpoints를 통해 실제 URL 패턴과 처리 로직을 연결(매핑)한다.
app.UseEndpoints(endpoints =>
{
    // 예시 1: "files/파일명.확장자" 형태의 요청을 처리
    // Eg: /files/sample.txt
    endpoints.Map("files/{filename}.{extension}", async context =>
    {
        // RouteValues 컬렉션에서 "filename"과 "extension" 매개변수 값을 꺼내온다.
        string? fileName = Convert.ToString(context.Request.RouteValues["filename"]);
        string? extension = Convert.ToString(context.Request.RouteValues["extension"]);

        // 꺼내온 값을 응답으로 보내준다.
        await context.Response.WriteAsync($"In files - Filename: {fileName}, Extension: {extension}");
    });

    // 예시 2: "employee/profile/직원이름" 형태의 요청을 처리
    // Eg: /employee/profile/john
    endpoints.Map("employee/profile/{EmployeeName}", async context =>
    {
        // RouteValues에서 값을 꺼낼 때 매개변수 이름은 대소문자를 구분하지 않는다. "employeename"으로도 OK.
        string? employeeName = Convert.ToString(context.Request.RouteValues["employeename"]);
        await context.Response.WriteAsync($"In Employee profile - Welcome, {employeeName}!");
    });
});

// 위에서 매핑된 주소가 하나도 없을 때 실행되는 기본 응답이다.
app.Run(async context => {
    await context.Response.WriteAsync($"The page you are looking for was not found at {context.Request.Path}");
});

app.Run();
```

#### 코드 해설 🤓

1. **`app.UseRouting()` & `app.UseEndpoints(...)`**: 라우팅 시스템을 사용하겠다고 선언하고, 어떤 URL 패턴이 들어왔을 때 어떤 코드를 실행할지 정의하는 부분이다.
2. **`endpoints.Map("files/{filename}.{extension}", ...)`**:
    - `files/` 와 `.` 은 **리터럴 텍스트**다. 요청 URL에 이 부분이 정확히 일치해야 한다.
    - `{filename}` 과 `{extension}` 은 **라우트 매개변수**다. 이 자리에는 어떤 텍스트든 올 수 있다.
    - 만약 브라우저에서 `http://localhost:port/files/mydocument.docx` 로 접속하면,
        - `filename` 변수에는 `"mydocument"`가 담기고,
        - `extension` 변수에는 `"docx"`가 담기게 된다.
3. **`context.Request.RouteValues["..."]`**: 이게 핵심이다. `RouteValues` 라는 주머니에 URL 패턴에서 정의한 매개변수 이름(Key)으로 실제 값(Value)이 담겨있다. 이 주머니에서 필요한 값을 이름으로 꺼내 쓰는 것이다.
    - 라우트 매개변수 이름은 **대소문자를 구분하지 않는다**. `{EmployeeName}`으로 정의했어도 `RouteValues["employeename"]`으로 꺼낼 수 있는 이유다.

---

### ☕️ Java Spring 의 경우

Spring에서는 `@PathVariable`이라는 어노테이션(Annotation)을 사용한다.

**C# / ASP.NET Core**

```csharp
endpoints.Map("employee/profile/{EmployeeName}", async context =>
{
    string? employeeName = Convert.ToString(context.Request.RouteValues["employeename"]);
    await context.Response.WriteAsync($"In Employee profile - Welcome, {employeeName}!");
});
```

**Java / Spring Boot**

```java
@RestController
public class EmployeeController {

    // "employee/profile/직원이름" 형태의 GET 요청을 이 메소드가 처리한다.
    @GetMapping("/employee/profile/{employeeName}")
    public String getEmployeeProfile(@PathVariable String employeeName) {
        // URL의 {employeeName} 부분이 메소드의 employeeName 파라미터로 바로 들어온다.
        return "In Employee profile - Welcome, " + employeeName + "!";
    }
}
```

#### 뭐가 비슷하고 뭐가 다른가?

- **공통점**: URL 경로에 `{...}`를 사용해서 변수를 선언하는 방식은 완전히 똑같다. 개념 자체가 동일하기 때문이다.
- **차이점**:
    - ASP.NET Core에서는 `HttpContext` 객체의 `Request.RouteValues`에서 직접 값을 꺼내는 방식이라면,
    - Spring은 `@PathVariable` 어노테이션이 마법을 부려서 URL의 값을 메소드의 파라미터로 **자동으로 주입(inject)**해준다. 훨씬 더 깔끔하고 직관적으로 보일 수 있다.

---

### 요약 정리! ✨

- URL에서 고정된 부분은 **리터럴 텍스트**, 변하는 부분은 **라우트 매개변수**라고 부른다.
- C# ASP.NET Core에서는 `{}`를 사용해 라우트 매개변수를 정의하고, `Request.RouteValues`에서 값을 꺼낸다.
- Java Spring에서는 똑같이 `{}`로 정의하고, `@PathVariable` 어노테이션을 사용해 메소드 파라미터로 값을 바로 받는다.