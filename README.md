# 🎨 디자인 패턴 (Design Patterns)

디자인 패턴은 **반복되는 설계 문제에 대한 검증된 해결 방식(Problem–Solution Template)** 이다.  
즉, “이 상황에서는 이렇게 설계하면 된다”라는 **구조적 사고 도구**다.
---

## 🧩 들어가기 전에 알아야 할 지식

디자인 패턴을 이해하기 위해서는  
다음 **객체지향 4대 원칙(OOP Four Principles)** 을 반드시 알고 있어야 한다.


## 🎯 객체지향 4대 원칙 (OOP Four Principles)

객체지향 프로그래밍(OOP)은  
**유연하고 확장 가능한 구조를 만들기 위한 설계 철학**이다.  

> 핵심 원칙: **상속(Inheritance)** · **추상화(Abstraction)** · **다형성(Polymorphism)** · **캡슐화(Encapsulation)**  

---

## 🧩 1️⃣ 상속 (Inheritance)

### 📘 정의  
> **상속은 기존 코드를 재사용하면서 일부 기능만 확장·재정의(Overriding)하는 메커니즘이다.**  
> 공통된 구조는 유지하고, 필요한 부분만 수정하여 새로운 동작을 만든다.

### 🔗 관련 개념
- **오버라이딩 (Overriding)** → 부모 메서드를 재정의하여 동작을 변경 (런타임 다형성 실현)  
- **오버로딩 (Overloading)** → 같은 이름의 메서드를 매개변수만 다르게 정의 (컴파일 타임 다형성 실현)

### 💻 예시
```java
class Animal {
    void speak() { System.out.println("동물이 소리를 냅니다."); }
}

class Dog extends Animal {
    @Override
    void speak() { System.out.println("멍멍!"); } // 오버라이딩
}

class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; } // 오버로딩
}
````

💬 **설명**
상속의 핵심은 **재사용**이며,
자식 클래스는 부모의 구조를 그대로 가져오면서 **일부 기능만 확장**하거나 **다르게 동작하게 재정의**한다.

---

## 🧩 2️⃣ 추상화 (Abstraction)

### 📘 정의

> **추상화는 공통된 행동만 남기고, 세부 구현은 감추는 것**이다.
> 즉, “무엇을 할지(What)”만 정의하고, “어떻게 할지(How)”는 하위 클래스가 결정한다.

### 🔗 관련 개념

* **인터페이스 (Interface)** → 구현 없이 동작의 명세만 정의 (행동의 계약)
* **추상 클래스 (Abstract Class)** → 공통 로직 일부를 구현하고, 세부는 자식이 완성

### 💻 예시

```java
interface Payment {
    void pay();
}

abstract class AbstractPayment implements Payment {
    void logStart() { System.out.println("결제 시작"); }
}

class CardPayment extends AbstractPayment {
    public void pay() {
        logStart();
        System.out.println("카드 결제 처리");
    }
}
```

💬 **설명**
인터페이스는 “행동의 틀”만,
추상 클래스는 “공통 로직”만 제공한다.
세부 구현은 각 클래스가 맡기 때문에 **확장과 유지보수가 쉬워진다.**

---

## 🧩 3️⃣ 다형성 (Polymorphism)

### 📘 정의

> **다형성은 같은 메서드 호출이지만, 실제 객체에 따라 다른 결과가 나오는 성질이다.**
> 즉, 하나의 인터페이스로 여러 구현체를 유연하게 다룰 수 있다.

### 🔗 관련 개념

* **인터페이스 기반 프로그래밍** → 상위 타입으로 코드를 작성하고, 실제 객체를 런타임에 주입
* **동적 바인딩 (Dynamic Binding)** → 실행 시점에 실제 객체 타입에 따라 호출 메서드가 결정됨

### 💻 예시

```java
interface FlyBehavior { void fly(); }

class FlyWithWings implements FlyBehavior {
    public void fly() { System.out.println("날개로 난다!"); }
}

class FlyNoWay implements FlyBehavior {
    public void fly() { System.out.println("못 난다!"); }
}

class Duck {
    private FlyBehavior flyBehavior;

    public void setFlyBehavior(FlyBehavior fb) { this.flyBehavior = fb; }
    public void performFly() { flyBehavior.fly(); }
}

// 실행 예시
Duck duck = new Duck();
duck.setFlyBehavior(new FlyWithWings());
duck.performFly(); // "날개로 난다!"
duck.setFlyBehavior(new FlyNoWay());
duck.performFly(); // "못 난다!"
```

💬 **설명**
`performFly()`는 하나지만, 실제 결과는 할당된 전략 객체에 따라 달라진다.
→ **“같은 메서드, 다른 동작”**, 이것이 다형성이다.

---

## 🧩 4️⃣ 캡슐화 (Encapsulation)

### 📘 정의

> **캡슐화는 데이터와 행위를 하나로 묶고, 외부에는 꼭 필요한 기능만 노출하는 것**이다.
> 즉, 내부 로직은 숨기고, 접근은 제한된 통로를 통해서만 허용한다.

### 🔗 관련 개념

* **접근 제어자 (Access Modifier)**

  * `private` → 내부 전용
  * `protected` → 상속 클래스 접근 가능
  * `public` → 외부 공개
  * `default` → 같은 패키지 내 접근 가능

### 💻 예시

```java
class User {
    private String password;

    public void setPassword(String pw) {
        if (pw.length() >= 8)
            this.password = pw;
        else
            System.out.println("비밀번호는 8자 이상이어야 합니다.");
    }

    public boolean login(String pw) {
        return this.password.equals(pw);
    }
}
```

💬 **설명**
`password`는 `private`으로 외부 접근을 차단하고,
`setPassword()`와 `login()` 같은 메서드만 공개한다.
→ 내부는 감추고, 외부엔 필요한 기능만 제공하는 **정보 은닉의 핵심 원칙.**

---

## 🧭 요약

| 원칙      | 정의 핵심                          | 대표 개념          | 핵심 포인트                |
| ------- | ------------------------------ | -------------- | --------------------- |
| **상속**  | 기존 코드를 **재사용**하며 **일부 기능만 확장** | 오버라이딩, 오버로딩    | 공통 구조 유지 + 필요한 부분만 수정 |
| **추상화** | 공통된 행동만 정의하고 세부 구현은 감춤         | 인터페이스, 추상클래스   | “무엇을 할지”만 정의          |
| **다형성** | 같은 호출, 다른 결과                   | 인터페이스 기반 프로그래밍 | 런타임 시점 행동 교체 가능       |
| **캡슐화** | 내부를 숨기고 필요한 기능만 공개             | 접근제어자, 정보은닉    | 정보 보호 + 변경 최소화        |

---
