# 생산자 소비자 문제2

## 1. Lock과 Condition 인터페이스

- `notify()`의 비효율 문제를 해결하려면, **생산자는 소비자만 깨우고, 소비자는 생산자만 깨워야** 한다.
- 이를 위해서는 **생산자 스레드가 대기하는 공간**과 **소비자 스레드가 대기하는 공간을 분리**할 필요가 있다.
- `java.util.concurrent.locks` 패키지의 **`Lock`과 `Condition` 인터페이스**를 사용하면 이를 구현할 수 있다.

### 1.1. Condition

- `Condition` 인터페이스는 `Lock` 객체와 연결된 **별도의 스레드 대기 공간(Wait Set)을 제공**한다.
- `Object.wait()`가 사용하는 내장 대기 집합과 달리, `Condition`은 `lock.newCondition()`을 통해 **직접, 그리고 여러 개를 생성**하여 사용할 수 있다.

### 1.2. condition.await()

- `Condition.await()` 메서드는 **`Object.wait()`와 유사한 기능**이다.
- 현재 스레드가 `Lock`을 반납하고, **해당 `Condition`의 대기 집합에서 `WAITING` 상태**로 대기하게 한다.

### 1.3. condition.signal()

- `Condition.signal()` 메서드는 **`Object.notify()`와 유사한 기능**이다.
- 해당 `Condition`의 대기 집합에서 **대기 중인 스레드 중 하나를 깨워** 락 획득 경쟁에 참여시킨다.
- 일반적으로 FIFO 순서대로 꺠운다. 자바 버전과 구현에 따라 달라질 수 있지만 대부분 Queue 구조로 구현하기 때문에 FIFO 순서로 꺠운다.
