# ItemProcessor

## 1. ItemProcessor

- `ItemProcessor`는 Spring Batch의 컴포넌트 중 하나로, **데이터 가공 또는 필터링**을 담당한다.
- **필수 요소는 아니며**, 필요할 때만 선언해서 사용할 수 있다.
- `ItemReader`에서 읽은 데이터를 `ItemProcessor`에서 변환하거나 필터링한 뒤 `ItemWriter`로 전달한다.
- 비즈니스 로직을 Reader/Writer와 **분리**할 수 있어 관심사 분리에 효과적이다.

### 1.1 주요 역할

- **변환(Transformation)**: 입력 객체를 원하는 형식으로 가공한다.
- **필터링(Filtering)**: 특정 조건을 만족하지 않으면 `null`을 반환하여 `ItemWriter`에 전달되지 않도록 한다.

## 2. ItemProcessor 인터페이스

```java
public interface ItemProcessor<I, O> {
    O process(I item) throws Exception;
}
```

- 제네릭 타입 `<I, O>`는 입력 타입과 출력 타입을 의미한다.
- `null`을 반환하면 해당 아이템은 writer로 전달되지 않는다.

## 3. CompositeItemProcessor (체이닝)

- 하나의 Processor로 모든 처리를 담당할 수 있지만, **여러 Processor를 체이닝**하고 싶은 경우 `CompositeItemProcessor`를 사용할 수 있다.
- 이는 **컴포지트 패턴**을 통해 여러 개의 ItemProcessor를 순차적으로 적용하는 방식이다.

### 3.1 예시

```java
// FooProcessor, BarProcessor 선언
public class Foo {}
public class Bar {
    public Bar(Foo foo) {}
}
public class Foobar {
    public Foobar(Bar bar) {}
}

public class FooProcessor implements ItemProcessor<Foo, Bar> {
    public Bar process(Foo foo) {
        return new Bar(foo);
    }
}

public class BarProcessor implements ItemProcessor<Bar, Foobar> {
    public Foobar process(Bar bar) {
        return new Foobar(bar);
    }
}

public class FoobarWriter implements ItemWriter<Foobar> {
    public void write(Chunk<? extends Foobar> items) {
        // write logic
    }
}
```

```java
// composite Processor 구성
@Bean
public CompositeItemProcessor<Foo, Foobar> compositeProcessor() {
    List<ItemProcessor<?, ?>> delegates = new ArrayList<>();
    delegates.add(new FooProcessor());
    delegates.add(new BarProcessor());

    CompositeItemProcessor<Foo, Foobar> processor = new CompositeItemProcessor<>();
    processor.setDelegates(delegates);
    return processor;
}
```

```java
// Step과 Job 설정
@Bean
public Step step1(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
    return new StepBuilder("step1", jobRepository)
        .<Foo, Foobar>chunk(2, transactionManager)
        .reader(fooReader())
        .processor(compositeProcessor())
        .writer(foobarWriter())
        .build();
}

@Bean
public Job ioSampleJob(JobRepository jobRepository, Step step1) {
    return new JobBuilder("ioSampleJob", jobRepository)
        .start(step1)
        .build();
}
```
