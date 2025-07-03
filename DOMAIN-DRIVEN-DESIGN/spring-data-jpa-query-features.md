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
