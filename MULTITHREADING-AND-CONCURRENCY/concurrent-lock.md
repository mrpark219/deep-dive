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

## 2. BLOCKED vs WAITING / TIMED_WAITING

- `BLOCKED`, `WAITING`, `TIMED_WAITING`은 모두 스레드가 실행을 멈추고 대기하는 '일시 중지' 상태이지만, **대기하는 원인과 빠져나오는 조건**에 명확한 차이가 있다.
- `WAITING` 상태에 시간 제한이 추가된 것이 `TIMED_WAITING` 상태이다.

| 구분              | `BLOCKED` (동기화 블록)               | `WAITING` / `TIMED_WAITING` (대기)                                                            |
| :---------------- | :------------------------------------ | :-------------------------------------------------------------------------------------------- |
| **발생 원인**     | **`synchronized`** 락(lock) 획득 대기 | `Object.wait()`, `Thread.join()`, `Thread.sleep()`, `LockSupport.park()` 등의 **명시적 호출** |
| **대기 대상**     | 다른 스레드의 **락(lock) 해제**       | 다른 스레드의 `notify()`, 작업 종료, 시간 초과 등 **특정 조건**                               |
| **인터럽트 반응** | **반응 안 함 (무시)**                 | **반응 함** (`InterruptedException` 발생)                                                     |
| **핵심 용도**     | 임계 영역(Critical Section) 접근 제어 | 스레드 간의 협력 및 조건부 대기                                                               |

- CPU 입장에서는 세 상태 모두 실행되지 않는다는 점은 동일하지만, **`BLOCKED`는 `synchronized` 전용**이고, **`WAITING` 계열은 더 능동적으로 제어**할 수 있는 대기 상태라는 차이가 있다.
