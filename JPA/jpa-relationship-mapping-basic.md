# 연관관계 매핑 기초

## 객체와 테이블의 연관관계 차이

### 테이블의 연관관계

- 테이블은 **외래 키(Foreign Key)** 를 사용해 연관된 데이터를 찾는다.
- `JOIN`을 통해 다른 테이블의 데이터를 조회한다.

### 객체의 연관관계

- 객체는 **참조(Reference)** 를 사용해 연관된 객체를 찾는다.
- 데이터베이스와 다르게, **객체 그래프 탐색이 가능**해야 한다.
- **SQL 중심이 아닌 객체 중심의 설계가 필요하다.**

## 연관관계 용어 정리

| 개념                   | 설명                                               |
| ---------------------- | -------------------------------------------------- |
| 방향(Direction)        | 단방향, 양방향                                     |
| 다중성(Multiplicity)   | 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:M) |
| 연관관계의 주인(Owner) | 양방향 관계에서 연관관계를 관리하는 주체           |

## 연관관계가 필요한 이유

### 테이블 중심 설계의 문제점

- 외래 키를 직접 관리해야 하며, SQL을 직접 작성해야 한다.
- 객체지향적이지 않다. → **객체 설계와 데이터베이스 설계 불일치**

### 객체 중심 설계의 장점

- 객체의 **참조(Reference)를 통해 연관 객체를 쉽게 조회** 가능하다.
- **객체 그래프 탐색**이 가능하다. (`member.getTeam().getName()`)
- 데이터베이스의 **외래 키를 직접 다루지 않아도 된다**.

## 단방향 연관관계

- 한 객체에서 다른 객체로만 참조하는 관계이다.
- 반대 방향에서는 참조할 수 없다.
- 테이블의 외래 키는 존재하지만, 객체 관점에서는 **한 방향으로만 관계를 탐색**할 수 있다.
- 조회가 한 방향으로만 필요한 경우, 단순한 관계를 유지하고 싶은 경우에 불필요한 복잡성을 줄이기 위해 사용한다.

### 단방향 연관관계 예제

```java
// Member 엔티티
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne // 다대일(N:1) 단방향 연관관계
    @JoinColumn(name = "team_id") // 외래 키 매핑
    private Team team;
}
```

```java
// Team 엔티티
@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;

    private String name;
}
```

```java
// 사용 예시

// 조회
Member findMember = em.find(Member.class, member.getId());

// 참조를 사용하여 연관관계 조회
Team findTeam = findMember.getTeam();

// 연관관계 수정
member.setTeam(newTeam);
```

```sql
-- 데이터베이스 테이블 구조
CREATE TABLE Member (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    team_id BIGINT,
    FOREIGN KEY (team_id) REFERENCES Team(id)
);

CREATE TABLE Team (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255)
);
```

## 양방향 연관관계와 연관관계의 주인

- 데이터베이스 테이블의 연관관계에서는 외래 키 하나만 있으면 양방향 연관관계가 된다. 양쪽으로 조인을 할 수 있기 때문이다.
  - **회원 ↔ 팀 (양방향)**
- 객체에서는 참조를 연관관계로 이어줄 두 객체에 각각 추가해야 한다. 따라서 객체의 양방향 연관관계는 **단방향 연관관계 2개로 이루어져 있다.**
  - **회원 → 팀 (단방향)**
  - **팀 → 회원 (단방향)**

따라서 객체 연관관계에서는 회원 객체 혹은 팀 객체 중 하나에서 **외래 키를 관리**해야 한다. (객체와 데이터베이스 테이블의 차이점)  
이때, **외래 키를 관리하는 객체를 연관관계의 주인(Owner)** 이라고 부른다.

### 연관관계의 주인(Owner)

- 두 객체 중 **하나를 연관관계의 주인**으로 지정한다.
- **연관관계의 주인만이 외래 키를 관리(등록, 수정)** 할 수 있다.
- 주인이 아닌 쪽은 읽기만 가능하다.
- 연관관계의 주인은 `mappedBy` 속성을 사용하지 않는다.
- 주인이 아닌 쪽은 `mappedBy` 속성으로 연관관계의 주인을 지정한다.
- **외래 키가 있는 곳을 주인으로 정한다.** (보통 `@ManyToOne`이 있는 곳)

### 양방향 연관관계 예제

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY) // 연관관계의 주인
    @JoinColumn(name = "team_id") // 외래 키 매핑
    private Team team;

    // 연관관계 편의 메서드
    public void setTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}
```

```java
@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team") // 주인이 아님 (읽기 전용)
    private List<Member> members = new ArrayList<>();
}
```

```java
// 팀 생성
Team team = new Team();
team.setName("Team A");
em.persist(team);

// 회원 생성
Member member = new Member();
member.setName("Member1");
member.setTeam(team); // 연관관계 설정
em.persist(member);

// 양방향 연관관계 확인
team.getMembers().forEach(m -> System.out.println(m.getName())); // Member1 출력
```

### 양방향 매핑 시 가장 많이 하는 실수

- 연관관계의 주인에 값을 입력하지 않는다.

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

// 역방향(주인이 아닌 방향)만 연관관계 설정
// mappedBy는 읽기 전용이므로 외래 키가 저장되지 않음
team.getMembers().add(member);
em.persist(member);
```

- 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정해야 한다.

`flush()`, `clear()` 후에 새롭게 데이터베이스에서 불러오면 변경된 정보가 새롭게 반영되지만, 한 객체에서만 변경하고 그대로 사용하면 정보가 다르게 보인다.
그렇기 때문에 연관관계 편의 메서드를 생성하는 것이 좋다. `set`으로 시작하는 함수명보다는 명확한 의미를 가진 함수명을 사용하는 것이 좋으며, 두 객체 모두가 아닌 한 객체에서만 만들어야 한다.

```java
public class Member {
    ...
    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this); // 연관관계 편의 메서드
    }
}
```

- 양방향 매핑 시에 무한 루프를 조심해야 한다.
  - `toString()`, Lombok의 `@Data`, `@ToString`, JSON 생성 라이브러리(Jackson 등) 사용 시 무한 루프가 발생할 수 있다.

### 양방향 연관관계 정리

- 단방향 매핑만으로 이미 연관관계 매핑은 완료된 것이다.
- 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것일 뿐이다.
- JPQL에서 역방향 탐색이 필요할 경우에만 추가하는 것이 좋다.
- 단방향 매핑을 먼저 적용하고, 필요한 경우에만 양방향을 추가한다. (테이블에는 영향을 주지 않는다.)

| 항목                    | 연관관계의 주인 (`@ManyToOne`) | 주인이 아닌 객체 (`@OneToMany`)  |
| ----------------------- | ------------------------------ | -------------------------------- |
| `mappedBy` 사용 여부    | ❌ 사용하지 않음               | ✅ 사용 (주인을 지정)            |
| 외래 키 관리 가능 여부  | ✅ 가능 (등록, 수정)           | ❌ 불가능 (읽기 전용)            |
| 데이터 변경 가능 여부   | ✅ 가능                        | ❌ 불가능                        |
| 객체 연관관계 설정 방법 | `member.setTeam(team);`        | `team.getMembers().add(member);` |
