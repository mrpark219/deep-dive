# 스프링 데이터 JPA를 이용한 조회 기능

## 1. CQRS

- **CQRS(Command Query Responsibility Segregation)** 는 시스템의 책임을 **명령(Command) 모델과 조회(Query) 모델로 분리**하는 패턴이다.
- **명령 모델**은 애그리거트와 같이 상태를 변경하는 비즈니스 로직을 수행하는 데 사용된다.
- **조회 모델**은 화면에 데이터를 보여주는 것과 같이, 데이터를 조회하는 로직을 수행하는 데 사용된다.
- 복잡한 도메인 모델은 주로 명령 모델로 활용되며, 정렬, 페이징, 다중 조건 검색 등은 조회 모델의 책임에 해당한다.

## 2. 검색을 위한 스펙

- 목록을 조회하는 기능은 다양한 검색 조건을 조합해야 할 때가 많다. 이때 **필요한 조합마다 `find` 메서드를 새로 정의**하는 것은, 조합의 수가 늘어날수록 관리해야 할 메서드가 폭발적으로 증가하기 때문에 좋은 방법이 아니다.
- 이렇게 다양한 검색 조건을 조합해야 할 때 사용할 수 있는 것이 바로 **스펙(Specification) 패턴**이다. 스펙은 **특정 조건을 충족하는지 검사하는 인터페이스**이며, 보통 아래와 같이 정의된다.

```java
public interface Specification<T> {
    public boolean isSatisfiedBy(T agg);
}
```

- `isSatisfiedBy()` 메서드는 검사 대상 객체(`agg`)가 **특정 조건을 충족하면 `true`를, 그렇지 않으면 `false`를 반환**한다. 리포지터리나 DAO는 이 스펙을 이용해 검색 대상을 걸러내는 데 사용한다.
- 하지만 실제로 이런 방식으로 스펙을 구현하는 경우는 거의 없다. 모든 애그리거트 객체를 메모리에 올린 뒤 하나씩 검사하는 것은 **현실적으로 불가능하며, 심각한 성능 문제**를 일으키기 때문이다.

## 3. 스프링 데이터 JPA를 이용한 스펙 구현

- 스프링 데이터 JPA는 **검색 조건을 표현하기 위한 `Specification` 인터페이스**를 제공하며, 아래와 같이 정의되어 있다.

```java
public interface Specification<T> extends Serializable {
    @Nullable
    Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder);
}
```

- `Specification` 인터페이스의 제네릭 타입 `T`는 JPA 엔티티 타입을 의미한다.
- 핵심 메서드인 `toPredicate()` 는 JPA의 **크리테리아(Criteria) API**를 이용해서 검색 조건을 표현하는 **`Predicate` 객체를 생성**하는 역할을 한다.
- 각각의 스펙을 별도의 클래스로 구현할 수도 있지만, 보통은 **별도의 유틸리티 클래스에 스펙을 생성하는 정적 메서드들을 모아두거나**, `Specification`이 함수형 인터페이스이므로 **람다식(Lambda Expressions)을 이용**하여 간결하게 구현한다.

## 4. 리포지터리/DAO에서 스펙 사용하기

- 스프링 데이터 JPA의 `JpaSpecificationExecutor` 인터페이스는 `Specification`을 파라미터로 받아 검색을 수행하는 `findAll()` 메서드를 제공한다.
- 이 메서드를 이용해 스펙을 충족하는 엔티티 목록을 아래와 같이 조회할 수 있다.

```java
// "board1" ID를 가진 Board를 검색하기 위한 스펙을 생성한다.
Specification<Board> spec = new BoardIdSpec("board1");

// 리포지터리에 스펙을 전달하여 검색 결과를 조회한다.
List<Board> results = boardRepository.findAll(spec);
```

## 5. 스펙 조합

- 스프링 데이터 JPA의 `Specification` 인터페이스는 **여러 스펙을 논리적으로 조합**할 수 있도록 다양한 디폴트(default) 및 정적(static) 메서드를 제공한다.
- **`and()`**
  - 두 스펙의 조건을 **모두 충족(AND)** 하는 새로운 스펙을 생성한다.
- **`or()`**
  - 두 스펙의 조건 중 **하나 이상을 충족(OR)** 하는 새로운 스펙을 생성한다.
- **`not()`**
  - 주어진 스펙의 **조건을 반전(NOT)** 시키는 새로운 스펙을 생성한다.
- **`where()`**
  - 정적 메서드로, 전달된 스펙이 `null`이면 아무 조건도 없는 스펙을, `null`이 아니면 전달된 스펙을 그대로 반환한다. 이는 **조건부로 스펙을 조합할 때 `NullPointerException`을 방지**하는 데 유용하다.

## 6. 정렬 지정하기

- 스프링 데이터 JPA에서는 크게 **두 가지 방법**으로 조회 결과의 정렬 순서를 지정할 수 있다.

### 6.1. 메서드 이름에 `OrderBy` 사용하기

- 쿼리 메서드의 이름에 `FindBy...OrderBy프로퍼티명Asc/Desc`와 같은 규칙을 사용하여 **정렬 순서를 직접 명시**할 수 있다.
- 이 방식은 구현이 매우 간단하지만, **정렬 기준이 두 개 이상이면 메서드 이름이 과도하게 길어지고**, 정렬 순서를 동적으로 변경할 수 없다는 단점이 있다.

### 6.2. `Sort` 객체 전달하기

- 더 유연한 방법은, 쿼리 메서드의 파라미터로 **`Sort` 객체를 전달**하여 정렬 순서를 동적으로 지정하는 것이다.
- `Sort` 객체의 `and()` 메서드를 사용하면, **여러 개의 정렬 조건을 연결**하여 복잡한 정렬 순서를 만들 수도 있다.

## 7. 페이징 처리하기

- 목록 기능에서 페이징 처리는 기본이다. 스프링 데이터 JPA는 **`Pageable` 인터페이스**를 이용하여 페이징을 간편하게 처리한다.
- 쿼리 메서드에 `Pageable` 타입의 파라미터를 추가하면, **스프링 데이터 JPA가 자동으로 페이징 기능을 적용**해준다.
- `Pageable`은 인터페이스이므로, 실제 객체는 `PageRequest.of()` 정적 메서드를 이용해 생성한다.
- `PageRequest.of()` 메서드에 `Sort` 객체를 함께 전달하면, 페이징 처리와 동시에 정렬도 수행할 수 있다.
- 페이징 관련 쿼리 메서드의 반환 타입으로 `Page<T>`를 사용하면, 조회된 데이터 목록뿐만 아니라 **전체 데이터 개수, 전체 페이지 수, 현재 페이지 번호** 등 다양한 페이징 관련 정보를 함께 얻을 수 있다.
- 만약 페이징 정보 없이 단순히 **상위 N개의 데이터만 가져오고 싶다면**, `Pageable`을 사용하는 대신 `findFirstN...` 또는 `findTopN...` 형식의 쿼리 메서드를 사용할 수도 있다.

## 8. 동적 인스턴스 생성

- JPA는 JPQL 쿼리 결과를 이용해 **임의의 객체를 동적으로 생성**할 수 있는 기능을 제공한다. 이는 주로 **조회 전용 모델(DTO 등)을 만드는 데 사용**된다.
- JPQL의 `SELECT` 절에서 `new` 키워드와 함께 **생성할 객체의 전체 클래스 이름을 명시**하고, 괄호 안에 생성자에 전달할 값들을 지정하는 방식으로 동작한다.
- 이 방식의 가장 큰 장점은, **JPQL의 객체 기준 쿼리 문법을 그대로 사용**하면서도, 즉시/지연 로딩과 같은 복잡한 고민 없이 **화면에 필요한 데이터만 정확히 조회**할 수 있다는 점이다.

## 9. 하이버네이트 @Subselect 사용

- 하이버네이트는 JPA 확장 기능으로 `@Subselect` 애너테이션을 제공한다.
- 이는 **복잡한 조회 쿼리의 결과를 하나의 엔티티처럼 매핑**할 수 있는 유용한 기능이다.
- `@Subselect`는 `SELECT` 쿼리 자체를 값으로 가진다. 하이버네이트는 이 쿼리의 실행 결과를, 마치 데이터베이스의 뷰(View)처럼, 가상의 테이블로 사용하여 엔티티와 매핑한다.
- 데이터베이스의 뷰처럼 `@Subselect`를 통해 매핑된 엔티티는 **수정할 수 없다**. 실수로 상태를 변경하더라도 DB에 반영되지 않도록, 함께 **`@Immutable` 애너테이션을 붙여주는 것**이 좋다.
