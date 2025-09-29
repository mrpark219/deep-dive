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

## 3. ReentrantLock

### 3.1 Lock 인터페이스

```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long var1, TimeUnit var3) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```

- **`Lock` 인터페이스**는 `synchronized` 키워드보다 더 유연하고 세밀한 제어가 가능한 **락(Lock)을 구현**하는 데 사용된다.
- 대표적인 구현체로는 **`ReentrantLock`**이 있다. `Lock` 인터페이스가 제공하는 락은 객체 내부의 모니터 락과는 다른 별개의 락이다.

#### void lock()

- **락을 획득**할 때까지 대기한다. 만약 다른 스레드가 이미 락을 획득했다면, 락이 풀릴 때까지 현재 스레드는 `WAITING` 상태가 된다. 이 메서드는 인터럽트에 응답하지 않는다.
- `WAITING` 상태에서 인터럽트가 발생하면 순간 대기 상태에서 빠져나와 `RUNNABLE` 상태가 되지만, `lock()` 메서드 안에서 해당 스레드에 다시 `WAITING` 상태로 강제 변경한다.

#### void lockInterruptibly()

- **인터럽트에 반응**하며 락을 획득할 때까지 대기한다.
- 만약 다른 스레드가 이미 락을 획득했다면 대기 상태가 되며, 대기 중에 인터럽트가 발생하면 `InterruptedException`을 던지고 락 획득을 포기한다.

#### boolean tryLock()

- 락을 획득할 수 있는지 **한 번만 시도하고 즉시 결과를 반환**한다.
- 락을 획득하면 `true`, 다른 스레드가 이미 락을 가지고 있어 실패하면 `false`를 반환하며, 대기하지 않는다.

#### boolean tryLock(long time, TimeUnit unit)

- **지정된 시간 동안만 락 획득을 시도**한다.
- 시간 내에 락을 획득하면 `true`, 시간이 지나도 획득하지 못하면 `false`를 반환한다. 대기 중 인터럽트에 반응한다.

#### void unlock()

- **획득했던 락을 해제**한다. 락을 해제하면 대기 중인 다른 스레드가 락을 획득할 수 있게 된다.
- 반드시 락을 획득한 스레드가 호출해야 하며, 그렇지 않으면 `IllegalMonitorStateException`이 발생할 수 있다.

#### Condition newCondition()

- `Lock`과 연결된 **`Condition` 객체를 생성하여 반환**한다.
- 이는 `Object` 클래스의 `wait()`, `notify()`와 유사하지만, 더 유연하고 강력한 스레드 간의 통신 기능을 제공한다.

### 3.2. 공정성 (Fairness)

- `Lock` 인터페이스의 대표 구현체인 `ReentrantLock`은 스레드가 락을 획득하는 순서에 대한 **공정성(Fairness) 모드**를 설정할 수 있다.

#### 비공정 모드 (Non-fair mode)

- `ReentrantLock`의 **기본 모드**이며, 락을 먼저 요청한 순서와 상관없이 락을 획득한다.
- **성능이 더 좋지만**, 특정 스레드가 오랫동안 락을 얻지 못하는 **기아(Starvation) 현상이 발생**할 수 있다.

#### 공정 모드 (Fair mode)

- `new ReentrantLock(true)`와 같이 생성자에 `true`를 전달하여 설정한다.
- **락을 요청한 순서대로 획득**하는 것을 보장하여 **기아 현상을 방지**한다.
- 하지만 스레드들을 관리하는 비용 때문에 **비공정 모드보다 성능이 저하**될 수 있다.

### 3.3. 주의 사항: finally 블록에서의 unlock() 호출

- 임계 영역 실행이 끝나면 **반드시 `unlock()`을 호출하여 락을 반납**해야 한다. 그렇지 않으면 다른 스레드들이 영원히 락을 얻지 못해 대기 상태에 빠질 수 있다.
- 따라서, **`lock.unlock()`은 반드시 `finally` 블록 안에서 호출**해야 한다.
- 이렇게 하면, `try` 블록 안에서 예외가 발생하거나 `return`을 통해 메서드가 중간에 종료되더라도, **`finally` 블록의 `unlock()` 호출은 반드시 보장**된다.
