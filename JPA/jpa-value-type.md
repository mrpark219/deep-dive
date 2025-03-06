# 값 타입

값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다. 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.

## 기본 값 타입

### JPA의 데이터 타입 분류

- 엔티티 타입
  - `@Entity`로 정의하는 객체이다.
  - 데이터가 변해도 **식별자를 통해 지속해서 추적 가능**하다.
  - 예를 들어, 회원 엔티티의 키나 나이 값이 변경되어도 **식별자로 계속 인식 가능**하다.
- 값 타입
  - 단순히 값으로 사용하는 데이터 타입이다.
  - `int`, `Integer`, `String`처럼 식별자가 없고 값만 존재하는 타입이다.
  - 값 타입은 변경되면 새로운 값으로 완전히 대체된다.
  - 예를 들어, 숫자 `100`을 `200`으로 변경하면 기존 `100`이 사라지고 `200`으로 교체된다.

### 값 타입의 분류

- 기본 값 타입
  - 자바 기본 타입(`int`, `double`)
  - 래퍼 클래스(`Integer`, `Long`)
  - `String`
- 임베디드 타입(복합 값 타입)
  - 사용자 정의 객체를 하나의 타입으로 사용하는 방식이다.
- 컬렉션 값 타입
  - 자바 컬렉션(`List`, `Set`)에 기본 값 타입이나 임베디드 타입을 저장하는 방식이다.

### 기본 값 타입 특징

- 생명 주기가 **엔티티에 의존적**이다
  - 예를 들어, 회원 엔티티가 삭제되면 이름, 나이 같은 값 타입 필드도 함께 삭제된다.
- 값 타입은 **공유해서는 안 된다**.
  - 값 타입이 공유되면 한 엔티티의 값을 변경할 때 다른 엔티티의 값도 함께 변경될 수 있다.
  - 예를 들어, 한 회원의 이름을 변경할 때 다른 회원의 이름까지 함께 변경되면 안 된다.
- 예제: `String name`, `int age`와 같은 값들이 기본 값 타입이다.

### 참고 - 자바의 기본 타입은 공유할 수 없다.

- 기본 타입(primitive type)은 항상 값을 복사하므로 공유할 수 없다.
  - `int`, `double`과 같은 기본 타입은 별도의 메모리 공간을 가지며, 참조를 공유하지 않는다.
- 래퍼 클래스(`Integer`, `Long`)와 `String`은 공유 가능하지만, 변경할 수 없다.
  - 래퍼 클래스와 `String`은 **참조값을 공유할 수 있지만, 내부 값을 변경할 방법이 없다.**
  - 따라서 안전하게 사용할 수 있다.

## 임베디드 타입(복합 값 타입)

- 새로운 값 타입을 직접 정의할 수 있는 기능이다.
- 기본 값 타입을 묶어서 사용할 수 있으며, 복합 값 타입이라고도 한다.
- `int`, `String`과 같은 값 타입과 유사하지만, 객체로 관리할 수 있다.

### 임베디드 타입 예시

- 회원 엔티티가 이름, 근무 시작일, 근무 종료일, 주소 정보를 가진다고 가정한다.

  ```mermaid
  classDiagram
    class Member {
        Long id
        String name
        Date startDate
        Date endDate
        String city
        String street
        String zipcode
    }
  ```

- 근무 시작일과 종료일을 하나의 값 타입(근무 기간)으로 묶을 수 있다.
- 도시, 번지, 우편 번호를 하나의 값 타입(주소)으로 묶을 수 있다.
- 이러한 방식으로 값 타입을 구성할 때 임베디드 타입을 사용한다.

  ```mermaid
  classDiagram
    class Member {
        Long id
        String name
        Period workPeriod
        Address homeAddress
    }

    class Period {
      Date startDate
      Date endDate
      isWork()
    }

    class Address {
      String city
      String street
      String zipcode
    }
  ```

### 임베디드 타입 예제 코드

```java
@Entity
public class Member {

  @Id @GeneratedValue
  private Long id;

  private String name;

  @Embedded
  private Period workPeriod;

  @Embedded
  private Address homeAddress;
```

```java
@Embeddable
public class Period {

  private LocalDateTime startDate;

  private LocalDateTime endDate;

  public boolean isWork() {
    // ...
  }
}
```

```java
@Embeddable
public class Address {

  private String city;

  private String street;

  private String zipCode;
}
```

### 임베디드 타입 사용법

- `@Embeddable`: 값 타입을 정의하는 클래스에 선언한다.
- `@Embedded`: 값을 사용하는 엔티티 필드에 선언한다.
- **기본 생성자가 필수**이다.

### 임베디드 타입의 장점

- 재사용이 가능하다.
- 관련된 값들을 논리적으로 묶어 응집도를 높일 수 있다.
- 값 타입 내부에 의미 있는 메서드를 추가할 수 있다.
  - Period.isWork()처럼 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수 있다.
- 임베디드 타입을 포함한 모든 값 타입은 값 타입을 소유한 엔티티의 생명주기를 의존한다.

### 임베디드 타입과 테이블 매핑

```mermaid
classDiagram
    class Member {
        <<Entity>>
        id : Long
        name : String

        -- Period --
        startDate : Date
        endDate : Date

        -- Address --
        city : String
        street : String
        zipcode : String
    }
```

```mermaid
erDiagram
    MEMBER {
        BIGINT ID PK
        VARCHAR NAME
        DATE STARTDATE
        DATE ENDATE
        VARCHAR CITY
        VARCHAR STREET
        VARCHAR ZIPCODE
    }
```

- 임베디드 타입은 엔티티의 일부이며, 별도의 테이블이 아닌 기존 엔티티 테이블에 매핑된다.
- 임베디드 타입을 사용하더라도 **테이블 구조는 변하지 않는다**.
- 객체를 더욱 세밀하게(`fine-grained`) 매핑할 수 있다.
- ORM 설계에서 테이블 수보다 클래스 수가 많아지는 것이 일반적이다.

### 임베디드 타입과 연관관계

- 임베디드 타입 내부에서 다른 엔티티와 연관관계를 맺을 수 있다.

```java
@Embeddable
public class Address {

  private String city;

  private String street;

  private String zipCode;

  @ManyToOne
    @JoinColumn(name = "country_id") // 연관관계 설정
    private Country country;
}
```

### @AttributeOverride 속성 재정의

- 같은 값 타입을 한 엔티티에서 여러 번 사용할 경우, 컬럼명이 중복될 수 있다.
- 이 문제를 해결하기 위해 **컬럼명을 재정의**해야 한다.
- `@AttributeOverride` 또는 `@AttributeOverrides`를 사용하여 컬럼명을 변경할 수 있다.

```java
@Entity
public class Member {

  @Id @GeneratedValue
  private Long id;

  private String name;

  @Embedded
  private Period workPeriod;

  @Embedded
  private Address homeAddress;

  @Embedded
  @AttributeOverrides({
    @AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
    @AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
    @AttributeOverride(name = "zipcode", column = @Column(name = "WORK_ZIPCODE"))
  })
  private Address workAddress;
```

### 임베디드 타입과 null

- 임베디드 타입의 필드 값이 `null`이면, 해당 타입이 매핑한 모든 컬럼 값도 `null`이 된다.

## 값 타입과 불변 객체

### 값 타입 공유 참조

- **임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험하다**.
- 공유하면 부작용(side effect)이 발생할 수 있다.
- 예를 들어, 회원1과 회원2가 같은 주소(Address) 객체를 참조하고 있다면, 회원1의 주소를 변경할 때 회원2의 주소도 함께 변경되는 문제가 발생할 수 있다.

```mermaid
flowchart TD
    A[희원1] --> C[주소]
    B[희원2] --> C[주소]
```

### 값 타입 복사

- 값 타입의 실제 인스턴스(객체)를 공유하는 것은 위험하다.
- 대신 **값(인스턴스)을 복사해서 사용**해야 한다.
- 객체를 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 방지할 수 있다.

```mermaid
flowchart LR
    A[희원1] -->|address| C[주소 - oldCity]
    B[희원2] -->|copyOfAddress| D[주소 - newCity]
```

### 객체 타입의 한계

- 항상 값을 복사해서 사용하면 공유 참조로 인한 부작용을 피할 수 있다.
- 하지만, **임베디드 타입처럼 직접 정의한 값 타입은 기본 타입이 아니라 객체 타입이므로 참조 값을 직접 대입하는 것을 막을 방법이 없다**.
  - 자바 기본 타입(int, double 등)은 값을 대입하면 복사된다.
  - 객체 타입은 참조 값을 대입하므로 공유 참조 문제가 발생할 수 있다.
- **객체의 공유 참조는 피할 수 없다**.

```java
// 기본 타입(primitive type)
int a = 10;
int b = a; // 기본 타입은 값을 복사
b = 4;
```

```java
// 객체 타입
Address a = new Address("old");
Address b = a; // 객체 타입은 참조를 전달
b.setCity("New");
```

### 불변 객체

- 객체 타입을 수정할 수 없게 만들면 부작용을 원천적으로 차단할 수 있다.
- **값 타입은 불변 객체(immutable object)로 설계**해야 한다.
- **불변 객체: 생성된 이후 절대 값을 변경할 수 없는 객체**이다.
- **생성자로만 값을 설정하고 수정자(Setter)를 만들지 않으면 된다**. 혹은 생성하고 private 접근 제어자로 지정한다.
  - Integer, String은 자바가 제공하는 대표적인 불변 객체이며, Setter가 없다.
- 불변이라는 작은 제약을 통해 부작용이라는 큰 문제를 막을 수 있다.
  - 값을 변경하려면 새로운 객체를 생성해야 하므로, 기존 객체가 변경되지 않아 안전하다.

## 값 타입의 비교

### 동일성(identity) 비교

- **인스턴스의 참조 값을 비교하는 방식**이다.
- **`==` 연산자를 사용하여 비교**한다.
- 같은 메모리 주소를 가리킬 때만 동일하다고 판단한다.

### 동등성(equivalence) 비교

- **인스턴스가 가지고 있는 값을 비교하는 방식**이다.
- **`equals()` 메서드를 사용하여 비교**한다.
  - JPA에서 프록시를 사용하는 경우, `equals()` 내부 구현 시 필드에 직접 접근하지 않고 getter를 호출해야 한다.  
    (프록시 객체는 실제 엔티티가 아니라 가짜 객체이므로, 직접 필드에 접근하면 원하는 값이 없을 수도 있다.)

### 값 타입의 비교

- **값 타입은 인스턴스가 달라도 내부 값이 같으면 같은 것으로 간주해야 한다.**
  - **값 타입은 `==` 연산자가 아니라 `a.equals(b)`를 사용해 동등성 비교를 해야 한다.**
- **값 타입의 `equals()` 메서드를 적절하게 재정의해야 한다.**
  - 보통 모든 필드를 포함하여 비교하도록 `equals()`를 구현한다.

```java
int a = 10;
int b = 10;

System.out.println("a == b: " + (a == b)); // true
```

```java
class Address {

  private String city;

  public test() {}

  public test(String city) {
    this.city = city;
  }

  @Override
  public boolean equals(Object o) {
    if(o == null || getClass() != o.getClass()) return false;
    test test = (test) o;
    return Objects.equals(city, test.city);
  }
}

Address a = new Address("TEST");
Address b = new Address("TEST");

System.out.println("a == b: " + (a == b)); // false
System.out.println("a.equals(b): " + (a.equals(b))); // true
```
