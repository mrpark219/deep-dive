# 고급 동기화 - concurrent.Lock

## 1. LockSupport

- `LockSupport`는 **스레드를 정지시키고 깨우는 기능을 제공**하는 매우 유용한 유틸리티 클래스이다.
- 이는 `synchronized`의 단점인 **'무한 대기' 문제를 해결**하고, 스레드를 더 세밀하게 제어할 수 있게 한다.
- `park()` 계열의 메서드는 스레드를 **`WAITING` 또는 `TIMED_WAITING` 상태**로 만들며, 이 상태의 스레드는 다른 스레드가 깨워주기 전까지 CPU 스케줄링에서 제외된다.

### 1.1. park()

- `park()` 메서드는 현재 스레드를 **`WAITING` 상태**로 만들어, 다른 스레드가 `unpark()`를 호출해줄 때까지 무기한 대기시킨다.

### 1.2. parkNanos(nanos)

- `parkNanos(nanos)` 메서드는 현재 스레드를 **지정된 나노초(nanoseconds) 시간 동안만 `TIMED_WAITING` 상태**로 만든다.
- 시간이 지나면 자동으로 `RUNNABLE` 상태로 돌아온다.

### 1.3. unpark(Thread thread)

- `unpark(Thread thread)` 메서드는 **`WAITING` 또는 `TIMED_WAITING` 상태에 있는 특정 대상 스레드를 즉시 `RUNNABLE` 상태로** 변경하여 깨우는 역할을 한다.
