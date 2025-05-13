# ItemWriter

## 1. ItemWriter

- `ItemWriter`는 Spring Batch에서 **데이터 출력**을 담당하는 컴포넌트이다.
- 출력 대상은 데이터베이스 저장, 파일 기록, API 전송 등 다양할 수 있다.
- `ItemReader` 또는 `ItemProcessor`를 통해 가공된 데이터를 **Chunk 단위로 받아 처리**한다.
- 일반적으로 I/O 작업은 비용이 크기 때문에, Chunk 단위로 **일괄 처리(batch processing)** 하여 성능을 최적화한다.

## 2. ItemWriter 인터페이스

```java
public interface ItemWriter<T> {
    void write(Chunk<? extends T> items) throws Exception;
}
```

- `write()`는 `ItemWriter`의 핵심 메서드이다.
- 이 메서드는 청크 단위로 전달된 데이터를 받아 한 번에 출력한다.
- 보통 데이터 저장, API 호출, 파일 쓰기 등의 작업이 이 메서드 내부에서 수행된다.
- `Chunk<? extends T>`는 처리 대상 객체 목록을 담고 있으며, 이들을 순회하거나 일괄로 처리할 수 있다.
