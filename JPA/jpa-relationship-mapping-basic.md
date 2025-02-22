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
