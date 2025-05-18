# 반복

## 1. Repeat

- 배치 작업은 본질적으로 **반복적인 처리**로 구성된다.
- 이 반복은 단순한 루프일 수도 있고, **예외 재시도나 특정 조건까지 반복 수행하는 전략적 작업**일 수도 있다.
- Spring Batch는 이러한 반복 작업을 일반화하여 처리하기 위해 `RepeatOperations` 인터페이스를 제공한다.
- 해당 인터페이스는 반복 제어, 완료 조건 설정, 예외 전략 등을 통합적으로 관리할 수 있도록 해준다.

## 2. RepeatTemplate

```java
public interface RepeatOperations {
    RepeatStatus iterate(RepeatCallback callback) throws RepeatException;
}
```

- `RepeatOperations`의 대표적인 구현체는 `RepeatTemplate`이며, **가장 단순하고 범용적인 반복 처리 도구**이다.
- `iterate()` 메서드는 반복을 수행하며, **`RepeatCallback`을 인자로 받아 반복마다 실행할 작업을 정의**한다.

```java
public interface RepeatCallback {
    RepeatStatus doInIteration(RepeatContext context) throws Exception;
}
```

- 반복 수행 중 **실제 작업을 정의하는 콜백 인터페이스**이다.
- 반복은 `RepeatStatus.CONTINUABLE` 또는 `RepeatStatus.FINISHED` 값을 반환하여 **반복 여부를 제어**한다.

```java
RepeatTemplate template = new RepeatTemplate();
template.setCompletionPolicy(new SimpleCompletionPolicy(2));  // 최대 2번 반복

template.iterate(new RepeatCallback() {
    public RepeatStatus doInIteration(RepeatContext context) {
        // 반복 수행할 작업 정의
        return RepeatStatus.CONTINUABLE; // 반복 계속
    }
});
```

- 위 예제는 최대 2번 반복하도록 설정된 `RepeatTemplate`을 사용하는 예이다.
- 내부 작업에서 CONTINUABLE을 리턴하더라도, `CompletionPolicy`에 의해 반복 횟수가 초과되면 자동으로 종료된다.

## 3. RepeatContext

```java
context.setAttribute("retryCount", 3);
Integer count = (Integer) context.getAttribute("retryCount");
```

- 반복 중 사용되는 **컨텍스트 정보 저장소** 역할을 한다.
- 반복 작업 동안 **일시적인 데이터나 상태를 보관**할 수 있는 attribute bag으로 작동한다.
- `RepeatCallback`의 인자로 전달되며, 반복 중 공유되는 데이터가 필요할 경우 유용하다.
- 일반적인 반복에서는 사용되지 않지만, **재시도 처리나 조건부 반복 제어** 등에 활용된다.

## 4. Completion Policies(완료 정책)

```java
RepeatTemplate template = new RepeatTemplate();
template.setCompletionPolicy(new SimpleCompletionPolicy(3)); // 최대 3회 반복
```

- `RepeatTemplate`의 `iterate()` 메서드가 **언제 반복을 종료할지**는 `CompletionPolicy`에 의해 결정된다.
- `CompletionPolicy`는 단순히 반복 종료 여부만 판단하는 것이 아니라, **`RepeatContext`의 생성과 상태 업데이트 책임도 함께 수행**한다.
- `RepeatTemplate`는 `CompletionPolicy`를 통해 `RepeatContext`를 생성하고, 이 컨텍스트를 반복 작업마다 `RepeatCallback`에 전달한다.
- `CompletionPolicy` 기본 구현체인 `SimpleCompletionPolicy`가 있다.
  - 고정된 반복 횟수를 지정할 수 있는 가장 간단한 구현체이다.
  - 생성자에 반복 횟수를 지정하면 해당 횟수만큼 반복 후 자동 종료된다.
