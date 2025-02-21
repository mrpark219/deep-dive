# 엔티티 매핑

## 엔티티 매핑 정리

| 매핑 대상              | 어노테이션                  |
| ---------------------- | --------------------------- |
| **객체와 테이블 매핑** | `@Entity`, `@Table`         |
| **필드와 컬럼 매핑**   | `@Column`                   |
| **기본 키 매핑**       | `@Id`                       |
| **연관관계 매핑**      | `@ManyToOne`, `@JoinColumn` |

## @Entity

- @Entity가 붙은 클래스는 JPA가 관리하는 엔티티 객체이다.
- 기본 생성자(파라미터가 없는 public 또는 protected 생성자)가 필수이다.
  - JPA는 리플렉션을 이용해 객체를 생성하고 관리하기 때문에 기본 생성자가 필요하다.
- 제약사항
  - final 클래스, enum, interface, inner class에는 @Entity를 사용할 수 없다.
  - 영속 필드(DB에 저장할 필드)에 final을 사용할 수 없다.

### @Entity 속성

| 속성     | 설명                                                                  |
| -------- | --------------------------------------------------------------------- |
| `name`   | JPA에서 사용할 엔티티 이름을 지정 <br> (기본값: 클래스 이름)          |
| `@Table` | 엔티티와 매핑할 **테이블명**을 지정 <br> (기본값: 클래스 이름과 동일) |

## 데이터베이스 스키마 자동 생성

### `hibernate.hbm2ddl.auto` 옵션

| 옵션          | 설명                                                       |
| ------------- | ---------------------------------------------------------- |
| `create`      | 기존 테이블을 삭제 후 다시 생성 (DROP + CREATE)            |
| `create-drop` | `create`와 동일하지만, 애플리케이션 종료 시 테이블을 삭제  |
| `update`      | 변경 사항만 반영 (운영 DB에서는 사용 금지)                 |
| `validate`    | 엔티티와 테이블이 정상적으로 매핑되었는지 검증 (변경 없음) |
| `none`        | 자동 생성 기능을 사용하지 않음                             |

### 주의 사항

- 운영 환경에서는 `create`, `create-drop`, `update` 사용 금지
- 환경별 권장 설정
  - 개발 초기 단계: `create` 또는 `update`
  - 테스트 서버: `update` 또는 `validate`
  - 스테이징 및 운영 서버: `validate` 또는 `none`
- DDL 생성 기능은 애플리케이션 실행 시 테이블을 생성할 때만 사용되며, JPA의 실행 로직에는 영향을 주지 않는다.
  - 예: 유니크 제약 조건 추가 등 DDL 관련 설정은 반영되지만, 런타임 동작에는 영향 없음

## 필드와 컬럼 매핑

### 매핑 어노테이션 정리

| 어노테이션    | 설명                                                   |
| ------------- | ------------------------------------------------------ |
| `@Column`     | 컬럼 매핑 (필드와 데이터베이스 컬럼을 매핑)            |
| `@Temporal`   | 날짜/시간 타입 매핑                                    |
| `@Enumerated` | `enum` 타입 매핑                                       |
| `@Lob`        | 대용량 데이터 (`BLOB`, `CLOB`) 매핑                    |
| `@Transient`  | 특정 필드를 컬럼에 매핑하지 않음 (JPA가 관리하지 않음) |

### @Column

| 속성                    | 설명                                                                                                                            | 기본값                                            |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| `name`                  | 필드와 매핑할 테이블의 컬럼 이름                                                                                                | 객체의 필드 이름                                  |
| `insertable, updatable` | 등록, 변경 가능 여부                                                                                                            | `TRUE`                                            |
| `nullable(DDL)`         | `null` 값 허용 여부 (`false`로 설정하면 `NOT NULL` 제약조건 추가됨)                                                             |                                                   |
| `unique(DDL)`           | 한 컬럼에 간단히 유니크 제약조건을 설정할 때 사용 (`@Table`의 `uniqueConstraints`와 유사하지만 이름 지정과 다중 컬럼 지정 불가) |                                                   |
| `columnDefinition(DDL)` | 데이터베이스 컬럼 정보를 직접 지정 (ex: `varchar(100) default 'EMPTY'`)                                                         | 필드의 자바 타입과 방언 정보를 사용하여 자동 설정 |
| `length(DDL)`           | 문자열 길이 제약 조건 (`String` 타입에서만 사용 가능)                                                                           | `255`                                             |
| `precision, scale(DDL)` | `BigDecimal`, `BigInteger` 타입에서 사용 (`precision`: 전체 자릿수, `scale`: 소수 자릿수)                                       | `precision=19, scale=2`                           |

## @Enumerated

자바 `enum` 타입을 매핑할 때 사용한다.  
`ORDINAL` 타입은 `enum` 값의 순서를 저장하므로, 중간에 순서가 바뀌면 데이터가 꼬일 수 있어 사용하면 안 된다.  
가능하면 `STRING` 타입을 사용하는 것이 안전하다.

| 속성    | 설명                                                                                                                              | 기본값             |
| ------- | --------------------------------------------------------------------------------------------------------------------------------- | ------------------ |
| `value` | - `EnumType.ORDINAL`: `enum` 순서를 데이터베이스에 저장 (사용 비추천) <br> - `EnumType.STRING`: `enum` 이름을 데이터베이스에 저장 | `EnumType.ORDINAL` |

## @Temporal

날짜 타입(`java.util.Date`, `java.util.Calendar`)을 매핑할 때 사용한다.  
`LocalDate`, `LocalDateTime`을 사용할 경우 `@Temporal`을 생략해도 된다.

### @Temporal 속성 정리

| 속성    | 설명                                                                                                                                                                                                                                                                                          | 기본값 |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| `value` | - **`TemporalType.DATE`**: 날짜, 데이터베이스 `DATE` 타입과 매핑 (예: `2025-02-21`) <br> - **`TemporalType.TIME`**: 시간, 데이터베이스 `TIME` 타입과 매핑 (예: `11:11:11`) <br> - **`TemporalType.TIMESTAMP`**: 날짜와 시간, 데이터베이스 `TIMESTAMP` 타입과 매핑 (예: `2025-02-21 11:11:11`) | 없음   |

## @Lob

데이터베이스 `BLOB`, `CLOB` 타입과 매핑할 때 사용한다.

- `@Lob`에는 지정할 수 있는 속성이 없다.
- 매핑하는 필드 타입이 문자면 `CLOB`, 그 외는 `BLOB`으로 매핑된다.

| 데이터 타입     | 매핑되는 데이터베이스 타입 |
| --------------- | -------------------------- |
| `String`        | `CLOB`                     |
| `char[]`        | `CLOB`                     |
| `java.sql.CLOB` | `CLOB`                     |
| `byte[]`        | `BLOB`                     |
| `java.sql.BLOB` | `BLOB`                     |

## @Transient

엔티티의 특정 필드를 데이터베이스에 매핑하지 않을 때 사용한다.

### 특징

- 해당 필드는 컬럼으로 매핑되지 않는다.
- 데이터베이스에 저장되지 않으며, 조회도 불가능하다.
- 주로 메모리상에서만 임시로 값을 저장할 때 사용된다.

## 기본 키 매핑

### 기본 키 매핑 어노테이션

- `@Id`: 기본 키(primary key)로 지정
- `@GeneratedValue`: 기본 키 값 자동 생성 전략 지정

### 기본 키 매핑 방법

| 전략      | 설명                                                 |
| --------- | ---------------------------------------------------- |
| 직접 할당 | `@Id`만 사용하여 기본 키를 직접 할당                 |
| 자동 생성 | `@GeneratedValue`를 사용하여 기본 키를 자동으로 생성 |

### 자동 생성 전략

| 전략       | 설명                                                                |
| ---------- | ------------------------------------------------------------------- |
| `IDENTITY` | 기본 키 생성을 데이터베이스에 위임 (ex. MySQL `AUTO_INCREMENT`)     |
| `SEQUENCE` | 데이터베이스 시퀀스를 사용 (ex. Oracle) - `@SequenceGenerator` 필요 |
| `TABLE`    | 키 생성용 테이블을 사용하여 기본 키 생성 - `@TableGenerator` 필요   |
| `AUTO`     | 방언(dialect)에 따라 자동으로 적절한 전략 선택 (기본값)             |

#### IDENTITY 전략

- 기본 키 생성을 **데이터베이스에 위임**
- 주로 **MySQL, PostgreSQL, SQL Server, DB2**에서 사용
- 데이터베이스의 `AUTO_INCREMENT` 기능을 활용하여 ID 값을 자동 생성

##### 동작 방식

- JPA는 일반적으로 **트랜잭션 커밋 시점**에 `INSERT SQL`을 실행
- 하지만 **IDENTITY 전략**은 `AUTO_INCREMENT` 방식으로 ID를 생성하므로,
  **`em.persist()` 시점에 즉시 `INSERT SQL`을 실행하고, 생성된 ID 값을 조회**
- 따라서 **영속성 컨텍스트에 저장될 때 ID가 결정됨**

##### 장점

- **설정이 간단**하고 데이터베이스의 기능을 활용할 수 있음
- **성능이 우수** (데이터베이스에서 자동으로 ID를 생성)

##### 단점

- **트랜잭션 롤백 시에도 ID가 증가** (AUTO_INCREMENT 특성상 재사용되지 않음)
- **배치 INSERT가 비효율적** (영속성 컨텍스트에 저장하기 전에 `INSERT SQL` 실행 필요)

##### 예제 코드

```java
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // IDENTITY 전략 사용
    private Long id;

    private String name;
}
```

#### SEQUENCE 전략

- 데이터베이스 **시퀀스(Sequence)**는 **유일한 값을 순서대로 생성**하는 객체
- **오라클, PostgreSQL, DB2, H2** 데이터베이스에서 사용 가능
- JPA에서 **SEQUENCE 전략**을 사용하면 `@SequenceGenerator`와 함께 설정해야 함

##### 동작 방식

1. **시퀀스 객체**를 사용하여 새로운 기본 키 값을 생성
2. `INSERT SQL`을 실행하기 전에 **먼저 시퀀스를 호출하여 ID 값을 가져옴**
3. 가져온 ID 값을 엔티티의 기본 키로 할당한 후 `INSERT SQL` 실행

##### 장점

- 데이터베이스에서 ID를 미리 조회 후 할당 → **배치 INSERT 가능**
- 트랜잭션 롤백 시에도 시퀀스 값 증가하지만, AUTO_INCREMENT보다는 제어가 쉬움

##### 단점

- 데이터베이스에 시퀀스 객체가 필요 (MySQL 등 지원하지 않는 DB에서는 사용 불가)

##### @SequenceGenerator

- **시퀀스 생성기(sequence generator)**를 정의할 때 사용

| 속성                | 설명                                                                                                     | 기본값               |
| ------------------- | -------------------------------------------------------------------------------------------------------- | -------------------- |
| `name`              | 식별자 생성기 이름 (필수)                                                                                | 필수                 |
| `sequenceName`      | 데이터베이스에 등록된 시퀀스 이름                                                                        | `hibernate_sequence` |
| `initialValue`      | DDL 생성 시에만 사용됨, 시퀀스 시작 값 지정                                                              | `1`                  |
| `allocationSize`    | 한 번 시퀀스 호출 시 증가하는 수 (성능 최적화 목적)<br> DB 시퀀스가 1씩 증가하면 이 값을 반드시 1로 설정 | 50                   |
| `catalog`, `schema` | 데이터베이스 catalog, schema 이름 지정                                                                   |                      |

---

##### 예제 코드

```java
@Entity
@SequenceGenerator(
    name = "member_seq_generator", // 시퀀스 생성기 이름
    sequenceName = "member_seq",   // 실제 데이터베이스 시퀀스 이름
    initialValue = 1,              // 시퀀스 시작 값
    allocationSize = 1             // 시퀀스 증가량 (DB 설정과 맞춰야 함)
)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member_seq_generator")
    private Long id;

    private String name;
}
```

#### Table 전략

- **키 생성 전용 테이블을 만들어** 데이터베이스 시퀀스를 **흉내내는 전략**

##### 장점

- 모든 데이터베이스에서 **사용 가능** (시퀀스 지원 여부에 관계없음)

##### 단점

- 성능이 낮음 (매번 키 생성 테이블을 조회해야 함)
- 동시성 제어가 필요할 수도 있음 (낮은 성능으로 인해 충돌 가능)

##### @TableGenerator

- 키 생성용 테이블을 정의할 때 사용

| 속성                   | 설명                                                  | 기본값              |
| ---------------------- | ----------------------------------------------------- | ------------------- |
| name                   | 식별자 생성기 이름                                    | 필수                |
| table                  | 키생성 테이블명                                       | hibernate_sequences |
| pkColumnName           | 시퀀스 컬럼명                                         | sequence_name       |
| valueColumnName        | 시퀀스 값 컬럼명                                      | next_val            |
| pkColumnValue          | 키로 사용할 값 이름                                   | 엔티티 이름         |
| initialValue           | 초기 값, 마지막으로 생성된 값이 기준이다.             | 0                   |
| allocationSize         | 시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨) | **50**              |
| catalog, schema        | 데이터베이스 catalog, schema 이름                     |                     |
| uniqueConstraints(DDL) | 유니크 제약 조건을 지정할 수 있다.                    |                     |

##### 예제 코드

```java
@Entity
@TableGenerator(
    name = "member_table_generator",  // 식별자 생성기 이름
    table = "my_sequences",           // 키 생성용 테이블명
    pkColumnName = "sequence_name",   // 시퀀스 이름을 저장할 컬럼명
    valueColumnName = "next_val",     // 시퀀스 값을 저장할 컬럼명
    pkColumnValue = "member_seq",     // 사용할 시퀀스 키 값
    initialValue = 1,                 // 초기 값
    allocationSize = 1                // 증가 값 (DB 설정과 맞춰야 함)
)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "member_table_generator")
    private Long id;

    private String name;
}
```

### 권장하는 식별자 전략

- 기본 키 제약 조건: `NULL 아님`, `유일`, `변경 불가`
- 자연 키 문제: 미래에도 위 조건을 만족하는 자연 키를 찾기 어려움
- 해결책: 대리 키(대체 키) 사용 권장
- 추천 방식: `Long 타입` + `대체 키` + `키 생성 전략 사용`
