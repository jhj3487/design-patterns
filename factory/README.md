
---

````markdown
# 🍕 Factory Pattern (Simple / Factory Method / Abstract Factory)

> “`new` 남발하는 코드에서 벗어나, 변화에 강한 객체 생성 구조를 만들자.”

이 문서는 **Head First Design Patterns – Factory 패턴 챕터**를 공부하면서 정리한 내용입니다.  

`Simple Factory`, `Factory Method`, `Abstract Factory`를 예제와 함께 한 번에 이해할 수 있도록 구성했습니다.

- 왜 `new`가 문제가 될 수 있는지  
- OCP(개방-폐쇄 원칙), 느슨한 결합과 어떤 관계가 있는지  
- 각 패턴이 어떤 상황에서 유리한지  

---

## 1. 왜 Factory 패턴이 필요한가?

일반적으로 객체를 만들 때 우리는 이렇게 작성합니다.

```java
Pizza pizza = new CheesePizza();
````

이 코드는 지금 현재는 아무 문제 없어 보이지만, **변화**가 필요한 순간 문제가 들어납니다.

* 새로운 타입이 추가될 때마다 `if / else`, `switch`가 늘어나거나 코스 수정 필요 함
* 클라이언트 코드가 구상 클래스(Concrete Class)를 직접 알아야 함
* 특정 구현에 맞춰 사용 시  강하게 결합 → **확장성이 떨어짐**
* 변경이 들어올 때마다 클라이언트 코드까지 수정 → **OCP(Open-Closed Principle) 위반**

즉,

> “변화 가능성이 높은 **객체 생성 책임**을 클라이언트 코드에서 분리해야 한다”

이 요구사항에서 나온 것이 바로 **Factory 계열 패턴**입니다.

---

## 2. Simple Factory (단순 팩토리)

### 2-1. 핵심 개념

* 객체 생성 로직을 한 클래스에 모아 **캡슐화**한 형태
* 흔히 “정적 팩토리(Static Factory)”처럼 사용
* 디자인 패턴 책에서 말하는 “정식 GoF 패턴”은 아니지만, 실무에서 자주 쓰이는 상용구 또는 기법

#### 구조

```
Client → SimpleFactory → Product

```

### 2-2. 예제 코드

```java
// Product 계층
public abstract class Pizza {
    public abstract void prepare();
    public abstract void bake();
}

public class CheesePizza extends Pizza {
    @Override
    public void prepare() { System.out.println("Preparing cheese pizza"); }

    @Override
    public void bake() { System.out.println("Baking cheese pizza"); }
}

public class PepperoniPizza extends Pizza {
    @Override
    public void prepare() { System.out.println("Preparing pepperoni pizza"); }

    @Override
    public void bake() { System.out.println("Baking pepperoni pizza"); }
}
```

```java
// Simple Factory
public class SimplePizzaFactory {

    public Pizza createPizza(String type) {
        if ("cheese".equals(type)) return new CheesePizza();
        if ("pepperoni".equals(type)) return new PepperoniPizza();

        throw new IllegalArgumentException("Unknown pizza type: " + type);
    }
}
```

```java
// Client
public class PizzaStore {

    private final SimplePizzaFactory factory;

    public PizzaStore(SimplePizzaFactory factory) {
        this.factory = factory;
    }

    public Pizza orderPizza(String type) {
        Pizza pizza = factory.createPizza(type); // 생성 책임을 팩토리에 위임
        pizza.prepare();
        pizza.bake();
        return pizza;
    }
}
```

### 2-3. 장단점 정리

**장점**

* 생성 로직이 한 곳으로 모여서 **관리 포인트가 명확함**
* 클라이언트가 `CheesePizza`, `PepperoniPizza` 같은 **구상 클래스 이름을 몰라도 됨**

**단점**

* 새로운 Pizza 타입이 늘어날 때마다 `SimplePizzaFactory`를 수정해야 함
  → OCP 완전 준수 어려움
* 하나의 큰 `if / else` 블록으로 성장할 수 있음

> 👉 **“변하는 부분(생성 로직)을 분리해서 캡슐화한 상용구 또는 기법” 정도로 이해하면 좋다.**

---

## 3. Factory Method (팩토리 메서드 패턴)

### 3-1. 핵심 개념

* 객체 생성 책임을 **서브클래스에게 위임**하는 패턴
* “어떤 구상 클래스를 생성할지는 서브클래스가 결정한다”
* **상속(Inheritance) + 메서드 오버라이딩** 기반 구조

#### 구조

```
Abstract Creator (PizzaStore)
 ├─ orderPizza()  ← 공통 로직 (템플릿)
 └─ createPizza() (abstract) ← 서브클래스에서 구현

Concrete Creator (NYPizzaStore, ChicagoPizzaStore)
 └─ createPizza() 오버라이드
```

### 3-2. 예제 코드

```java
// Product 계층은 Simple Factory와 동일
public abstract class Pizza {
    public abstract void prepare();
    public abstract void bake();
}
```

```java
// Creator (추상 클래스)
public abstract class PizzaStore {

    public Pizza orderPizza(String type) {
        // 공통 플로우는 상위 클래스가 정의 (템플릿 메서드 패턴 느낌)
        Pizza pizza = createPizza(type); // 구체 객체 생성은 서브클래스에게 위임
        pizza.prepare();
        pizza.bake();
        return pizza;
    }

    // 팩토리 메서드
    protected abstract Pizza createPizza(String type);
}
```

```java
// Concrete Creator
public class NYPizzaStore extends PizzaStore {

    @Override
    protected Pizza createPizza(String type) {
        if ("cheese".equals(type)) return new NYCheesePizza();
        if ("pepperoni".equals(type)) return new NYPepperoniPizza();
        throw new IllegalArgumentException("Unknown pizza type: " + type);
    }
}
```

```java
// Concrete Creator
public class ChicagoPizzaStore extends PizzaStore {

    @Override
    protected Pizza createPizza(String type) {
        if ("cheese".equals(type)) return new ChicagoCheesePizza();
        if ("pepperoni".equals(type)) return new ChicagoPepperoniPizza();
        throw new IllegalArgumentException("Unknown pizza type: " + type);
    }
}
```


### 3-3. 장단점 정리

**장점**

* 새로운 피자 스타일(뉴욕, 시카고 등) 추가 시
  → 새로운 `PizzaStore` 서브클래스만 추가하면 됨
  → 기존 상위 클래스 코드는 변경하지 않음 → **OCP 준수**
* 클라이언트는 `orderPizza()`만 호출하면 됨
  → 내부에서 어떤 구상 클래스가 생성되는지 모름 → **느슨한 결합(Loose Coupling)**

**단점**

* 상속 구조로 인해 클래스 수가 늘어남
* Simple Factory보다 구조 이해가 어렵고 설계가 무거워질 수 있음

> 👉 **“생성 책임을 서브클래스에게 넘겨서 확장을 통해 변화에 대응하는 패턴”**이라고 보면 된다.

---

## 4. Abstract Factory (추상 팩토리 패턴)

### 4-1. 핵심 개념

* **관련된 제품군(Product Family)을 생성하기 위한 인터페이스**를 제공하는 패턴
* 예: 피자 하나를 만들 때도 치즈, 도우, 소스 등 여러 재료(제품군)를 함께 사용
* 지역/환경/스타일에 따라 “제품군 전체를 통째로 바꾸고 싶을 때” 유용

Factory Method가

> “어떤 피자를 만들지”에 대한 생성 책임 분리라면,

Abstract Factory는

> “그 피자에 들어갈 재료들을 어떤 스타일로 구성할지”에 대한 책임 분리.

### 4-2. 인터페이스 정의

```java
// 재료 팩토리 인터페이스 (제품군 생성 인터페이스)
public interface IngredientFactory {
    Cheese createCheese();
    Dough createDough();
    Sauce createSauce();
}
```

지역별 구현:

```java
public class NYIngredientFactory implements IngredientFactory {

    @Override
    public Cheese createCheese() {
        return new ReggianoCheese();
    }

    @Override
    public Dough createDough() {
        return new ThinCrustDough();
    }

    @Override
    public Sauce createSauce() {
        return new MarinaraSauce();
    }
}

public class ChicagoIngredientFactory implements IngredientFactory {

    @Override
    public Cheese createCheese() {
        return new MozzarellaCheese();
    }

    @Override
    public Dough createDough() {
        return new ThickCrustDough();
    }

    @Override
    public Sauce createSauce() {
        return new PlumTomatoSauce();
    }
}
```

피자 클래스에서 사용:

```java
public class CheesePizza extends Pizza {

    private final IngredientFactory ingredientFactory;

    public CheesePizza(IngredientFactory ingredientFactory) {
        this.ingredientFactory = ingredientFactory;
    }

    @Override
    public void prepare() {
        System.out.println("Preparing cheese pizza with specific ingredients...");
        this.cheese = ingredientFactory.createCheese();
        this.dough = ingredientFactory.createDough();
        this.sauce = ingredientFactory.createSauce();
    }
}
```

### 4-3. 장단점 정리

**장점**

* 지역/환경/컨텍스트에 따라 **제품군 전체 교체** 가능
  (예: 뉴욕 스타일 재료 ↔ 시카고 스타일 재료)
* 클라이언트는 구상 클래스가 아니라 **팩토리 인터페이스**에만 의존
  → 느슨한 결합
  → DIP(Dependency Inversion Principle)를 준수한 구조

**단점**

* 구조가 복잡하고 클래스/인터페이스가 많아짐
* 작은 규모 프로젝트에서는 과한 설계가 될 수도 있음

> 👉 **“관련된 객체들(제품군)을 생성 및 교체 시 사용하는 패턴”**이라고 이해하면 된다.

---

## 5. Factory 패턴 간 비교

| 패턴               | 초점                         | 구현 방식              | 확장 전략             | 특징 요약          |
| ---------------- | -------------------------- | ------------------ | ----------------- | -------------- |
| Simple Factory   | 생성 로직 캡슐화                  | 단일 클래스 + `if/else` | 클래스 수정(변경)        | 구조 단순, OCP 불완전 |
| Factory Method   | 생성 책임을 서브클래스에 위임           | 상속 + 메서드 오버라이드     | 서브클래스 추가(확장)      | OCP 준수, 느슨한 결합 |
| Abstract Factory | 관련된 제품군(Product Family) 생성 | 인터페이스 + 조합         | 팩토리 구현체 교체(환경 교체) | 제품군 단위 교체 가능   |

핵심 포인트는 다음 한 줄로 정리할 수 있다.

> **Simple → Factory Method → Abstract Factory로 갈수록
> 객체 생성에 대한 추상화 수준과 유연성이 높아진다.**

---

## 7. 마무리 정리

Factory 계열 패턴의 공통된 목표는 하나입니다.

> **“객체 생성 책임을 분리해서, 변화에 강하고 확장 가능한 구조를 만드는 것”**

그 과정에서 자연스럽게

* 추상화(Abstraction)
* 캡슐화(Encapsulation)
* 느슨한 결합(Loose Coupling)
* OCP, DIP 같은 설계 원칙

이 적용됩니다.

실무에서는

* 처음에는 Simple Factory 수준으로 시작했다가
* 복잡도가 올라가면 Factory Method 또는 Abstract Factory로 점진적으로 리팩토링하는 패턴이 많이 보입니다.

---
