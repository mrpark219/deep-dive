# 객체와 자료 구조

## 1. 자료 추상화

```java
// 구현 1
public class Point {
    public double x;
    public double y;
}
```

```java
// 구현 2
public interface Point {
    double getX();
    double getY();
    void setCartesian(double x, double y);
    double getR();
    double getTheta();
    void setPolar(double r, double theta);
}
```

- 구현 1은 **구현을 외부로 노출**한다. 즉, 객체의 내부 구조를 그대로 드러낸다.
- 구현 2는 **구현을 완전히 숨긴다**. 인터페이스를 통해 접근만 허용하며, 내부 구현 방식은 감춰져 있다.
- 구현 1에서 변수를 `private`으로 선언하더라도, 단순히 각각에 대해 getter와 setter를 제공한다면 **구현을 노출하는 것과 다름없다**.
- 구현 2는 단순한 자료 구조 이상을 표현하며, 클래스의 **메서드를 통해 접근 정책을 강제**한다.
- 변수 사이에 접근자를 끼워 넣는다고 구현이 자동으로 감춰지는 것이 아니다.
- 진정한 캡슐화는 **추상화**를 통해 가능하다.
- 추상 인터페이스를 제공하고, 사용자가 내부 구현을 알 필요 없이 핵심 동작만을 조작할 수 있어야 **진정한 의미의 클래스**라 할 수 있다.

## 2. 자료/객체 비대칭

- 객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 제공한다.
- 자료 구조는 자료를 그대로 공개하며, 별다른 함수를 제공하지 않는다.
- 객체와 자료 구조는 **정반대의 개념**이다.
- **자료 구조**를 사용하는 **절차적인 코드**는 기존 자료 구조를 변경하지 않고 **새로운 함수를 쉽게 추가**할 수 있다. 하지만 **새로운 자료 구조를 추가**하는 것은 어렵다.
- 반면, **객체 지향 코드**는 기존 클래스를 변경하지 않고 **새로운 클래스를 쉽게 추가**할 수 있다. 그러나 **새로운 함수를 추가**하는 것은 어렵다.
- 정리:
  - **절차적 코드**는 **데이터 중심**, 새 함수 추가는 쉬우나 새 객체 타입 추가는 어려움
  - **객체지향 코드**는 **동작 중심**, 새 객체 타입 추가는 쉬우나 새 함수 추가는 어려움

## 3. 디미터 법칙

- **디미터 법칙(Law of Demeter)** 은 **모듈은 자신이 조작하는 객체의 내부 구조를 몰라야 한다**는 원칙이다.
- 객체는 자료를 숨기고 **행동(함수)** 만을 외부에 제공해야 한다.
- 디미터 법칙에 따르면 클래스 `C`의 메서드 `f`는 아래와 같은 대상의 메서드만 호출해야 한다:
  - 클래스 `C` 자신
  - `f`가 생성한 객체
  - `f`의 인수로 넘어온 객체
  - `C`의 인스턴스 변수에 저장된 객체

### 3.1 기차 충돌 (Train Wreck)

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

- 위 코드는 여러 객체가 **점(dot)** 으로 연결된 형태로, 마치 기차처럼 이어져 있어 **기차 충돌(train wreck)** 이라 부른다.
- 문제는 하나의 함수가 여러 객체의 **내부 구조를 너무 많이 알고 있다는 것**이다.
- 이는 디미터 법칙을 위반할 가능성이 높다. 예를 들어, `ctxt`, `Options`, `ScratchDir`이 모두 객체라면 내부 구조를 숨겨야 하므로 위반이다.
- 하지만 만약 이것들이 단순한 **자료 구조(struct-like)**라면, 법칙을 위반했다고 보긴 어렵다.

#### 더 나은 방식

```java
final String outputDir = ctxt.options.scratchDir.absoluteDir;
```

- 위처럼 작성하면 자료 구조라는 사실을 분명히 드러낸다. 따라서 **디미터 법칙을 적용하지 않아도 된다.**

### 3.2 잡종 구조

- **객체와 자료 구조의 혼합형**을 **잡종 구조**라고 한다.
- 잡종 구조는 일부는 객체처럼 행동하고, 일부는 자료처럼 행동한다.
- 예: 공개 변수도 있으면서 의미 있는 함수도 함께 있는 구조
- 이런 구조는 유지보수성이 낮고, **함수나 자료 구조 모두를 쉽게 확장할 수 없다.**

### 3.3 구조체 감추기

- 만약 `ctxt`, `options`, `scratchDir`이 진짜 **객체**라면, 다음과 같이 작성해서는 안 된다:

```java
ctxt.getOptions().getScratchDir().getAbsolutePath();
```

- 객체라면 내부 구조를 **숨기고** 외부에는 **동작만 제공**해야 한다.
- 따라서 아래와 같이 작성하는 것이 이상적이다:

```java
final String outputDir = ctxt.createTempFilePath();
```

- 즉, **원하는 작업의 의미를 담은 메서드를 객체에게 위임**함으로써 구조를 숨기고 인터페이스만 드러내는 것이 좋다.

## 4. 자료 전달 객체

- 자료 구조체의 전형적인 형태는 **공개 변수만 있고 함수가 없는 클래스**다.
- 이런 구조를 흔히 **자료 전달 객체(Data Transfer Object, DTO)** 라고 부른다.
- DTO는 보통 **데이터베이스에서 조회한 가공되지 않은 정보를 애플리케이션 코드에서 사용하기 위한 용도**로 쓰인다.
- 일반적인 형태는 **빈(Bean)** 구조(인수가 없는 생성자가 있고, 직렬화가 가능하고, 각 private 필드에 대해서 getter와 setter를 가진 Java 객체)이며, **비공개 변수와 이를 조회/설정하는 Getter/Setter 함수**를 갖는다.

```java
public class UserDTO {
    private Long id;
    private String name;
    private String email;

    // 생성자
    public UserDTO(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // getter / setter
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

### 4.1 활성 레코드 (Active Record)

- **활성 레코드(Active Record)** 는 DTO의 **특수한 형태**다.
- **공개 변수** 또는 **Getter/Setter가 있는 비공개 변수**를 가지며, 여기에 **save, find 등 데이터베이스 연산을 수행하는 메서드**가 함께 존재한다.
- 이러한 구조는 **데이터베이스 테이블이나 외부 소스에서 읽어온 결과를 표현**하는 데 적합하다.
- 하지만 이 구조에 **비즈니스 규칙을 담는 메서드를 추가**하면 문제가 생긴다.
  - 이로 인해 **자료 구조와 객체 역할이 섞이는 잡종 구조**가 되어 유지보수가 어려워진다.
- 해결 방법:
  - 활성 레코드는 **자료 구조로만 사용**한다.
  - **비즈니스 규칙은 별도의 객체**에 위임하고, 이 객체는 내부 자료를 숨기고 동작만을 공개한다.

```java
@Entity
@Table(name = "users")
public class User {

    @Id @GeneratedValue
    private Long id;
    private String name;
    private String email;

    // 기본 생성자
    public User() {}

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    // Getter/Setter
    // ...

    // Active Record 스타일 메서드
    public void save(EntityManager em) {
        if (this.id == null) {
            em.persist(this);
        } else {
            em.merge(this);
        }
    }

    public static User findById(EntityManager em, Long id) {
        return em.find(User.class, id);
    }

    public void delete(EntityManager em) {
        em.remove(em.contains(this) ? this : em.merge(this));
    }
}
```

## 5. 마무리

- **객체(Object)** 는 **동작을 외부에 공개**하고, **자료는 숨긴다.**
  - 기존 동작을 변경하지 않고 **새 객체 타입을 추가하기 쉽다.**
  - 반면 **새 동작을 추가하려면** 객체 구조를 고쳐야 하므로 **어렵다.**
- **자료 구조(Data Structure)** 는 **자료를 노출**하고, **동작은 거의 없다.**
  - 기존 함수에 **새 자료 구조를 추가하기 어렵지만,**
  - 기존 자료 구조에 **새 동작을 추가하기는 쉽다.**

> ✔ 시스템을 구현할 때 **어떤 유연성이 필요한지**에 따라 적합한 방식을 선택해야 한다.
>
> - **새로운 자료 구조(타입)를 자주 추가해야 한다면 → 객체 지향 설계가 적합하다.**
> - **새로운 동작(기능)을 자주 추가해야 한다면 → 자료 구조와 절차적인 설계가 적합하다.**

- 중요한 것은 **선입견 없이 문제의 본질을 파악하고**, 그에 맞는 **최적의 해결책**을 선택하는 일이다.
