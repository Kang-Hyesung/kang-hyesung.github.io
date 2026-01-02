---
title: Java 생성자 다중 정의(Overloading)
date: 2025-12-30 21:01 +0900
author: hyesung
description:
categories: JAVA
---
## 1. 생성자 다중정의(Constructor Overloading)란?

Java에서 클래스는 객체(Object)를 기술하는 명세서와 같다. 이 클래스를 통해 실제 메모리에 객체(인스턴스)를 생성할 때 가장 먼저 호출되는 것이 바로 **생성자(Constructor)**다.

일반 메서드와 마찬가지로 생성자 또한 매개변수의 개수나 타입을 다르게 하여 여러 개를 정의할 수 있으며, 이를 **생성자 다중정의(Overloading)**라고 한다.

### 왜 필요한가?

객체를 생성하는 상황은 다양하다. 어떤 경우에는 필수 정보만으로 객체를 만들어야 하고, 어떤 경우에는 모든 세부 정보를 포함해서 만들어야 할 수도 있다. 생성자 다중정의는 이러한 다양한 초기화 요구사항을 유연하게 처리할 수 있도록 돕는다.

* **기본 생성자:** 아무런 정보 없이 객체를 기본 상태로 생성할 때 사용
* **매개변수가 있는 생성자:** 객체 생성과 동시에 특정 데이터로 초기화가 필요할 때 사용

---

## 2. 중복 코드 제거: `this()`의 활용

생성자를 여러 개 만들다 보면 필연적으로 **초기화 코드의 중복**이 발생한다. 예를 들어, 멤버 필드의 초기화 로직이 모든 생성자에 똑같이 들어가 있다면, 추후 로직 수정 시 모든 생성자를 찾아다니며 수정해야 하는 비효율이 발생한다.

이때 사용하는 것이 바로 `this()`다. `this()`는 **같은 클래스 내의 다른 생성자를 호출**하는 역할을 한다.

### 효율적인 초기화 전략

강의에서는 **공통 초기화 로직을 하나의 생성자(주로 기본 생성자)에 집중**시키고, 다른 생성자에서 `this()`를 통해 이를 호출하는 방식을 권장한다.

1. **공통 코드 집중:** 모든 생성자가 공유해야 할 초기화 코드를 하나의 생성자(예: 기본 생성자)에 작성한다.
2. **호출 위임:** 다른 생성자들은 `this()`를 사용하여 그 공통 생성자를 먼저 호출한 뒤, 자신만의 추가적인 초기화 작업을 수행한다.

> **주의:** 생성자 내부에서 다른 생성자를 호출할 때는 반드시 **첫 줄**에 작성해야 한다. 그렇지 않으면 컴파일 에러가 발생한다.
{: .prompt-warning }

---

## 3. 실무 예제: UserData 클래스

강의에서 다룬 `UserData` 클래스를 기반으로, 생성자 다중정의와 `this()` 활용 패턴을 적용한 코드를 살펴본다.

```java
public class UserData {
    // 캡슐화를 위해 멤버 필드는 private으로 선언
    private String name;
    private int age;

    // 1. 기본 생성자
    public UserData() {
        // 공통 초기화 로직이 있다면 이곳에 작성
        System.out.println("[Log] 기본 생성자(UserData()) 호출됨");
        // 예: 객체 생성 시 공통적으로 수행해야 할 로깅이나 기본값 설정
    }

    // 2. 다중 정의된 생성자 (이름과 나이를 입력받음)
    public UserData(String name, int age) {
        // this()를 이용해 기본 생성자를 먼저 호출
        this(); 
        
        System.out.println("[Log] UserData(String, int) 호출됨");
        
        // 해당 생성자만의 추가 초기화 작업 수행
        this.name = name;
        this.age = age;
    }

    // Getter 메서드
    public String getName() {
        return name;
    }
}

// 메인 실행 클래스
public class Main {
    public static void main(String[] args) {
        // new 연산자를 통해 객체 생성 시점에 적절한 생성자가 호출됨
        System.out.println("--- 객체 생성 시작 ---");
        UserData user = new UserData("Hoon", 10);
        System.out.println("--- 객체 생성 완료 ---");
    }
}

```

### 코드 실행 흐름 분석

위 코드를 실행하면 다음과 같은 순서로 동작한다.

1. `new UserData("Hoon", 10)` 호출.
2. 매개변수가 있는 생성자 `UserData(String, int)` 진입.
3. 첫 줄의 `this()`를 만나 **기본 생성자** `UserData()`로 이동.
4. 기본 생성자 내부 코드 실행 (`[Log] 기본 생성자...` 출력).
5. 다시 원래 생성자로 복귀하여 나머지 코드 실행 (`[Log] UserData(String, int)...` 출력 및 필드 초기화).

---

> **💡 Deep Dive: 내부 동작 원리 (Call Stack & Bytecode)**
> 단순히 코드가 위에서 아래로 흐르는 것처럼 보이지만, JVM 내부에서는 **호출 스택(Call Stack)**이 생성되고 소멸하는 과정이 일어난다.
> 1. **객체 생성의 3단계:** `new` 키워드를 사용하면 JVM은 크게 세 가지 동작을 수행한다.
> * **메모리 할당:** 힙(Heap) 영역에 객체를 위한 메모리 공간을 확보하고 0(또는 null)으로 초기화한다.
> * **인스턴스 멤버 초기화:** 명시적으로 선언된 필드 값들이 있다면 이때 설정된다.
> * **생성자 실행 (`<init>`):** 마지막으로 사용자가 정의한 생성자 코드가 실행된다.
> 
> 
> 2. **호출 스택(Call Stack)의 동작:**
> `this()`를 사용하여 생성자를 연결하면, 마치 재귀 함수나 일반 메서드 호출처럼 **스택 프레임(Stack Frame)**이 쌓인다.
> * `UserData(String, int)`가 스택에 올라감(Push).
> * `this()` 호출로 인해 `UserData()`가 그 위에 올라감(Push).
> * `UserData()` 실행 완료 후 스택에서 제거(Pop).
> * `UserData(String, int)`의 나머지 코드가 실행되고 완료 후 제거(Pop).
> 
> 
> 3. **왜 `this()`는 첫 줄이어야 하는가?**
> 객체지향 원칙상, 자식 클래스나 현재 클래스의 구체적인 초기화가 이루어지기 전에 **상위(부모) 클래스나 공통의 기본적인 초기화가 먼저 완료**되어야 '완전한 객체' 상태를 보장할 수 있기 때문이다. Java 컴파일러는 이 순서를 강제하여 초기화되지 않은 필드에 접근하는 등의 오류를 원천 차단한다.
> 
> 

---

## 💡 Quiz: 학습 내용 확인하기

<details>
<summary><strong>Q1. 생성자 내에서 다른 생성자를 호출할 때 사용하는 키워드와, 사용할 때 주의할 점은 무엇인가?</strong></summary>





<strong>정답:</strong> <code>this()</code> 키워드를 사용하며, 반드시 생성자의 <strong>첫 번째 줄(Statement)</strong>에 작성해야 한다.
</details>

<details>
<summary><strong>Q2. 생성자 다중정의(Overloading)를 사용하는 주된 목적은 무엇인가?</strong></summary>





<strong>정답:</strong> 객체 생성 시 다양한 초기화 데이터(매개변수)를 받아들일 수 있도록 하여, <strong>객체 생성의 유연성과 편의성을 제공</strong>하기 위함이다.
</details>

<details>
<summary><strong>Q3. (심화) `new UserData(&quot;Test&quot;, 20)`를 호출했을 때, 힙(Heap) 메모리 할당과 생성자 실행 중 무엇이 먼저 일어나는가?</strong></summary>





<strong>정답:</strong> <strong>힙 메모리 할당</strong>이 먼저 일어난다. JVM은 먼저 객체를 저장할 메모리 공간을 확보하고 0으로 초기화한 뒤, 그 후에 생성자(<code>&lt;init&gt;</code> 메서드)를 호출하여 개발자가 작성한 초기화 로직을 수행한다.
</details>