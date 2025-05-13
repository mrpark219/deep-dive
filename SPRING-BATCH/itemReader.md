# ItemReader

## 1. ItemReader

- `ItemReader`는 **Spring Batch에서 데이터를 읽어오는 역할**을 수행한다.
- 한 번 호출할 때마다 **하나의 데이터를 반환**하며, 더 이상 읽을 데이터가 없으면 `null`을 반환한다.
- 데이터 소스는 데이터베이스, 파일, XML, JSON 등 다양하며, Spring Batch에서는 이를 위한 **다양한 기본 구현체를 제공**한다.
- 만약 기본 구현체로 지원되지 않는 특수한 데이터 소스가 있다면, **사용자가 직접 구현한(Custom) ItemReader를 정의**할 수도 있다.

👉 공식 문서에서 [Spring Batch가 제공하는 ItemReader 목록](https://docs.spring.io/spring-batch/reference/appendix.html#listOfReadersAndWriters)을 확인할 수 있다.

## 2. ItemReader Interface

```java
public interface ItemReader<T> {
    T read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException;
}
```

- `read()` 메서드는 `ItemReader`의 핵심 메서드이다.
- 이 메서드는 **한 번 호출될 때마다 데이터를 하나씩 반환**한다.
- 더 이상 읽을 데이터가 없다면 `null`을 반환하여 Chunk 반복을 종료한다.
- 일반적으로 도메인 객체 하나를 리턴하도록 설계된다.

## 3. 사용 예시

예를 들어 CSV 파일에서 회원 정보를 읽는다면 `ItemReader<Member>` 형태로 구성되고, `read()` 메서드는 한 줄씩 읽어 `Member` 객체로 반환한다.

- 반복 호출 흐름:
  ```
  read() → Member1
  read() → Member2
  ...
  read() → null (종료)
  ```
