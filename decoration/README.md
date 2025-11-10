```markdown
# 🎨 Decorator Pattern (데코레이터 패턴)

> **TL;DR**  
데코레이터 패턴은 **컴포넌트와 데코레이터 두 구성요소**로 이루어지며,  
객체의 기능을 **동적으로 추가**할 수 있는 **상속보다 유연한 확장 방식**이다.  
즉, **상속 대신 구성(Composition)과 위임(Delegation)** 으로 기능을 겹겹이 쌓고,  
모든 객체가 **같은 컴포넌트 인터페이스**를 공유하여 **OCP(확장에는 열려, 변경에는 닫혀)** 원칙을 실현한다.



---

## 목차
1. [정의](#정의-definition)
2. [언제 쓰나](#언제-쓰나-when-to-use)
3. [구조](#구조-structure)
4. [동작 방식](#동작-방식-how-it-works)
5. [장점과 단점](#장점과-단점-pros--cons)
6. [예시 1: 알림 시스템 (순수 자바)](#예시-1-알림-시스템-순수-자바)
7. [예시 2: Java I/O Stream 체인](#예시-2-java-io-stream-체인)
8. [순서 민감성 & 흔한 실수](#순서-민감성--흔한-실수)
9. [관련 패턴 비교](#관련-패턴-비교)
10. [체크리스트](#체크리스트)
11. [부록: 간단 UML (PlantUML)](#부록-간단-uml-plantuml)

---

## 정의 (Definition)

**데코레이터(Decorator) 패턴**은 **객체의 책임(기능)을 런타임에 동적으로 추가**하는 패턴이다.  
상속 대신 **구성(객체 포함)** 과 **위임(내부 객체에게 작업을 맡김)** 을 사용하며,  
**기존 코드를 수정하지 않고 확장**할 수 있게 하여 **OCP**를 실천한다.

---

## 언제 쓰나 (When to Use)

- 기능을 **선택적/가변적으로 조합**해야 한다.
- **런타임에** 기능을 붙였다 뗐다 해야 한다.
- 상속으로 조합하면 **클래스 폭발**이 발생할 것 같다.
- 기존 코드를 **수정하지 않고** 확장해야 한다.

---

## 구조 (Structure)

데코레이터는 아래 4요소로 구성된다.

| 요소 | 역할 | Java I/O 예시 |
|---|---|---|
| **Component**         | 공통 인터페이스/추상 클래스 | `InputStream` |
| **ConcreteComponent** | 실제 핵심 기능           | `FileInputStream`, `ByteArrayInputStream` |
| **Decorator**         | `Component`를 구현하고 **내부에 `Component`를 보관**(구성)하여 호출을 **위임**하는 추상 클래스 | `FilterInputStream` |
| **ConcreteDecorator** | 부가기능을 실제로 덧입힘    | `BufferedInputStream`, `DataInputStream`, `PushbackInputStream` 등 |

**텍스트 UML**
```markdown
```
Client
  → Component (interface/abstract)
       ↑                 ↑
       │                 │
 ConcreteComponent   Decorator (abstract, has-a Component)
                          ↑
                          │
                ConcreteDecoratorA / B / ...
```
```

---

## 동작 방식 (How it Works)

1. **클라이언트**는 `Component` 타입으로만 다룬다.
2. **ConcreteComponent**가 기본 기능을 수행한다.
3. **Decorator**는 생성자에서 `Component`를 받아 **필드에 보관**한다. (구성)
4. 데코레이터의 메서드는 **내부 `Component`에 위임**하고, **전/후 처리**로 부가기능을 추가한다.
5. 여러 데코레이터를 **체인처럼 중첩**하여 조합한다.

> 예: `new BufferedInputStream(new DataInputStream(new FileInputStream("data.bin")))`

---

## 장점과 단점 (Pros & Cons)

### ✔ 장점
- **OCP 준수**: 기존 코드 수정 없이 확장.
- **조합 유연성**: 다양한 데코레이터 조합으로 기능 구성.
- **상속 남용 방지**: 약결합 구조.
- **런타임 유연성**: 실행 중에도 래핑 변경 가능.

### ⚠ 단점
- **인스턴스 증가**: 체인이 길어질수록 객체 수 증가.
- **디버깅 복잡성**: 호출 경로가 깊어짐.
- **순서 민감성**: 데코레이터 적용 순서에 따라 결과/성능 달라짐.

---

## 예시 1: 알림 시스템 (순수 자바)

**의도**: 기본 알림을 다양한 채널(SMS, Slack, Email 등)로 **동적으로 확장**한다.  
**핵심**: **구성 필드**(슈퍼타입 `Notifier`)에 **위임**하여 기능을 겹겹이 추가.

```java
// Component
interface Notifier {
    void send(String message);
}

// ConcreteComponent
class BasicNotifier implements Notifier {
    @Override
    public void send(String message) {
        System.out.println("[Basic] " + message);
    }
}

// Decorator (구성 + 위임의 핵심)
abstract class NotifierDecorator implements Notifier {
    protected final Notifier component; // 구성(Composition): 내부에 Component 보관

    protected NotifierDecorator(Notifier component) {
        this.component = component;
    }
}

// ConcreteDecorators
class SmsDecorator extends NotifierDecorator {
    public SmsDecorator(Notifier component) { super(component); }

    @Override
    public void send(String message) {
        component.send(message);                 // 위임(Delegation)
        System.out.println("[SMS] " + message);  // 부가기능
    }
}

class SlackDecorator extends NotifierDecorator {
    public SlackDecorator(Notifier component) { super(component); }

    @Override
    public void send(String message) {
        component.send(message);                   // 위임(Delegation)
        System.out.println("[Slack] " + message);  // 부가기능
    }
}

// 사용 예
public class Main {
    public static void main(String[] args) {
        Notifier notifier =
            new SlackDecorator(               // 바깥쪽 래퍼
                new SmsDecorator(             // 중간 래퍼
                    new BasicNotifier()       // 안쪽 핵심
                )
            );

        notifier.send("빌드 실패 알림");
    }
}
````

**실행 결과**

```
[Basic] 빌드 실패 알림
[SMS] 빌드 실패 알림
[Slack] 빌드 실패 알림
```

**작동 원리(콜 체인)**

1. `SlackDecorator.send` → 2) `SmsDecorator.send` → 3) `BasicNotifier.send` → (복귀) `SmsDecorator` 후처리 → (복귀) `SlackDecorator` 후처리

> 모두 `Notifier` 타입이므로 **타입 일관성**이 유지되어 자유롭게 **중첩(래핑)** 가능하다.

---

## 예시 2: Java I/O Stream 체인

```java
try (DataInputStream in =
        new DataInputStream(
          new BufferedInputStream(
            new FileInputStream("data.bin")))) {

    int v = in.readInt(); // Buffered(성능↑) + Data(타입 단위 읽기) 기능이 겹겹이 적용
}
```

| 계층                | 클래스                                                                                   | 역할                               |
| ----------------- | --------------------------------------------------------------------------------------- | --------------------------------  |
| Component         | `InputStream`                                                                           | 공통 인터페이스                       |
| ConcreteComponent | `FileInputStream`                                                                       | 실제 파일에서 바이트 읽기               |
| Decorator         | `FilterInputStream`                                                                     | 모든 데코레이터의 상위 추상              |
| ConcreteDecorator | `BufferedInputStream`, `DataInputStream`, `PushbackInputStream`, `CheckedInputStream` 등 | 버퍼링/타입 읽기/푸시백/체크섬 등 기능 추가 |

---

## 순서 민감성 & 흔한 실수

* **순서 민감성**:

  * 예) `압축 → 암호화` vs `암호화 → 압축` 은 결과/성능이 다르다.
  * I/O에서도 버퍼링을 먼저 둘지, 데이터 변환을 먼저 둘지에 따라 성능과 사용 편의가 달라진다.
* **클래스가 동적으로 바뀌는 게 아님**:

  * **객체의 행동 구성이 런타임에 바뀌는 것**이다. (래핑 교체/추가)
* **상속과 혼동 금지**:

  * 상속은 컴파일타임 고정, 기반 클래스 변경에 취약.
  * 데코레이터는 **구성 + 위임**으로 유연성을 확보.

---

## 관련 패턴 비교

| 패턴            | 목적                 | 차이점                          |
| ------------- | -------------------  | ---------------------------- |
| **Strategy**  | 알고리즘(행위) 자체를 교체 | “무엇을 하느냐”를 바꿈                |
| **Decorator** | 기능을 덧입혀 확장       | “어떻게 확장하느냐”를 겹겹이 조합          |
| **Adapter**   | 인터페이스 변환         | 호환성 확보, 기능 확장 목적 아님          |
| **Proxy**     | 접근 제어/원격/지연 로딩  | 주 목적이 **간접화**(대리), 부가기능과는 구분 |

---

## 체크리스트

* [ ] `Component / ConcreteComponent / Decorator / ConcreteDecorator` 구분이 명확한가?
* [ ] 데코레이터가 **같은 슈퍼타입**을 구현하고 **내부에 Component 필드**를 갖는가?
* [ ] 메서드 구현에서 **위임(내부 component 호출)** 후 **전/후 처리**가 적용되는가?
* [ ] **순서 민감성**이 필요한 조합에서 순서를 의도적으로 정했는가?
* [ ] 기존 코드 수정 없이 새로운 데코레이터 추가로 **OCP**를 달성했는가?

---
## 🏁 요약
> 데코레이터 패턴은 “상속의 한계를 넘는 유연한 확장 방식”이다.  
> **구성(Composition)** 과 **위임(Delegation)** 을 통해 기능을 동적으로 조합하고,  
> **OCP** 원칙을 실천하여 유지보수성과 확장성을 모두 확보한다.
```