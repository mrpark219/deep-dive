# 프록시와 연관관계 정리

## 프록시

### 프록시가 필요한 이유

- 회원을 조회할 때, 팀 정보가 필요하지 않다면 팀 데이터를 즉시 가져오지 않고, 나중에 필요할 때만 데이터베이스에서 조회하여 **불필요한 리소스 낭비를 줄일 수 있다.**
- 프록시를 사용하면 **실제 엔티티 대신 가짜(프록시) 객체를 먼저 반환하고, 해당 객체가 처음 사용될 때 데이터베이스에서 실제 데이터를 가져오는 방식**으로 동작한다.

### `em.find()` VS `em.getReference()`

| 메서드              | 동작 방식                                          |
| ------------------- | -------------------------------------------------- |
| `em.find()`         | 데이터베이스를 조회하여 실제 엔티티를 반환한다.    |
| `em.getReference()` | 데이터베이스 조회를 미루는 프록시 객체를 반환한다. |

- `em.find()`는 즉시 데이터베이스에서 엔티티를 조회하는 반면, `em.getReference()`는 프록시 객체를 반환하고, 실제 데이터가 필요할 때까지 데이터베이스 조회를 미룬다.

```mermaid
classDiagram
    class Client {
    }

    class Proxy {
        Entity target = null
        getId()
        getName()
    }

    Client --> Proxy : em.getReference()
```

### 프록시 객체 특징

- 실제 클래스를 상속받아 만들어진다.
- 실제 클래스와 겉모양이 동일하다. (`instanceof`로 확인 가능)
- 처음 사용할 때 **한 번만 초기화**된다. 초기화 후에는 실제 객체를 참조한다.
- 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 된다.(이론상)

```mermaid
classDiagram
    class Entity {
        id
        name
        getId()
        getName()
    }

    class Proxy {
        getId()
        getName()
    }

    Entity <|-- Proxy : 상속
```

- 실제 객체를 참조(target)로 보관하며, 필요할 때 데이터베이스에서 조회한다.
- 프록시 객체를 호출하면, 내부적으로 실제 객체의 메서드를 실행한다.

```mermaid
classDiagram
    class Entity {
        id
        name
        getId()
        getName()
    }

    class Proxy {
        Entity target
        getId()
        getName()
    }

    Proxy --|> Entity : 위임(delegate)
```

### 프록시 객체의 초기화 과정

1. 클라이언트가 `memberProxy.getName()`을 호출한다.
   - `memberProxy`는 아직 실제 엔티티를 로드하지 않은 상태이다.
2. 프록시가 **영속성 컨텍스트에 초기화 요청을 보낸다.**
3. **영속성 컨텍스트에서 1차 캐시 확인 및 데이터베이스 조회를 수행한다.**
4. 데이터베이스에서 **실제 엔티티 데이터를 반환한다.**
5. **실제 엔티티를 생성하고 프록시 객체의 `target`에 연결한다.**
6. **프록시 객체가 실제 엔티티의 `getName()`을 실행한다.**

```mermaid
classDiagram
    class Client {
    }

    class MemberProxy {
        +getId()
        +getName()
        -Member target
    }

    class Member {
        -id
        -name
        +getId()
        +getName()
    }

    class 영속성컨텍스트 {
    }

    class DB {
    }

    Client --> MemberProxy : 1.요청 (getName)
    MemberProxy --> 영속성컨텍스트 : 2.초기화 요청
    영속성컨텍스트 --> DB : 3.DB 조회
    DB --> 영속성컨텍스트 : 4.데이터 반환
    영속성컨텍스트 --> Member : 5.실제 Entity 생성
    MemberProxy --> Member : 6.target.getName()
```

## 프록시 객체 사용 시 주의할 점

- **프록시 객체는 실제 엔티티로 변경되지 않는다.**
  - 초기화되더라도 여전히 프록시 객체이며, 내부적으로 실제 엔티티를 참조할 뿐이다.
- **프록시 객체는 원본 엔티티를 상속받는다.**
  - `==` 비교를 하면 실패할 수 있으므로, `instanceof`를 사용해야 한다.
- **영속성 컨텍스트에 같은 엔티티가 존재하면 `em.getReference()` 호출 시 실제 엔티티를 반환한다.**
  - 같은 트랜잭션 내에서는 `==` 비교 시 `true`가 나와야 하기 때문이다.
- **영속성 컨텍스트에서 분리된(detached) 상태에서 프록시를 초기화하면 예외가 발생한다.**
  - 데이터베이스에서 데이터를 조회할 수 없기 때문이다.

### 프록시 확인

| 확인 방법                                       | 설명                                             |
| ----------------------------------------------- | ------------------------------------------------ |
| `emf.getPersistenceUnitUtil().isLoaded(entity)` | 프록시 인스턴스의 초기화 여부를 확인한다.        |
| `entity.getClass().getName()`                   | 프록시 객체의 클래스명을 출력하여 확인한다.      |
| `org.hibernate.Hibernate.initialize(entity)`    | 프록시 객체를 강제로 초기화한다. (JPA 표준 아님) |

- JPA 표준에는 프록시 강제 초기화 기능이 없으며, 메서드를 호출하면 자동으로 초기화된다.

## 즉시 로딩과 지연 로딩

### 지연 로딩(LAZY)

"[프록시가 필요한 이유](#프록시가-필요한-이유)"를 해결하기 위해 **지연 로딩(LAZY)** 을 사용한다.

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY) // 지연 로딩 설정
    @JoinColumn(name = "team_id")
    private Team team;
}
```

```mermaid
graph LR
    db[(database)] -->|1.로딩| member((member))
    member -->|2.지연로딩 LAZY| team((프록시 team 엔티티))

    style team stroke-dasharray:5,5
```

- `em.find()`를 통해 `member`를 불러올 때, `team`의 패치 타입이 `LAZY`로 설정되어 있으면 **가짜 프록시 객체**를 가져온다.
- 그 뒤 `team`을 실제 사용하는 시점에 **프록시가 초기화되면서 DB에서 데이터를 조회**한다.

## 즉시 로딩(EAGER)

Member와 Team을 자주 함께 사용하는 경우 **즉시 로딩(EAGER)**을 설정할 수 있다.

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.EAGER) // 즉시 로딩 설정
    @JoinColumn(name = "team_id")
    private Team team;
}
```

```mermaid
graph LR
    db[(database)] -->|1.로딩| member((member))
    member -->|2.즉시로딩 EAGER| team((team - 실제 team 엔티티))
```

- `em.find()`를 통해 `member`를 불러올 때, `team`의 패치 타입이 `EAGER`로 설정되어 있으면 **실제 team 엔티티를 함께 가져온다.**
- JPA 구현체는 가능하면 **JOIN을 사용하여 SQL 한 번에 데이터를 조회**한다.

### 프록시와 즉시 로딩 주의

- **지연 로딩만 사용**해야 한다.(특히 실무에서)
  - 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생한다.
- **즉시 로딩은 JPQL에서 `N + 1 문제`를 일으킨다.**
- 즉시 로딩 대신에 **`JPQL fetch 조인`이나 `엔티티 그래프 기능`을 사용**해야 한다.
- **`@ManyToOne`, `@OneToOne`은 기본이 즉시 로딩이기 때문에 `LAZY`로 설정해야 한다.**
- **`@OneToMany`, `@ManyToMany`는 기본이 지연 로딩이다.**

### N + 1 문제

`memeber` 엔티티를 조회할 때 `team`이 즉시 로딩(EAGER)로 설정되어 있다면, `SELECT * FROM member`이 1번 실행되고, member의 수(N)만큼 추가적인 team 조회 쿼리(N번 실행)가 발생하여 N + 1 문제가 발생한다.

```sql
SELECT * FROM member; -- 1번 실행
SELECT * FROM team WHERE team_id = ?; -- 멤버 수(N)만큼 실행
```
