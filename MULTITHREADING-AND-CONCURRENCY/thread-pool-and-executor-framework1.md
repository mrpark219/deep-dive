# 스레드 풀과 Executor 프레임워크1

## 1. 스레드를 직접 사용할 때의 문제점

- 스레드를 직접 생성해서 사용하면, **성능, 관리, 불편함**이라는 세 가지 주요 문제점이 있다.

### 1.1. 스레드 생성 비용으로 인한 성능 문제

- 스레드를 생성하는 작업은 단순히 자바 객체를 만드는 것과는 비교할 수 없을 정도로 **무거운 작업**이다. 그 이유는 다음과 같다.
  - **메모리 할당**: 각 스레드는 자신만의 호출 스택(Call Stack)을 위한 메모리(보통 1MB 이상)를 할당받아야 한다.
  - **운영체제 자원 사용**: 스레드 생성은 운영체제 커널 수준에서 시스템 콜(System Call)을 통해 처리되므로, CPU와 메모리 자원을 소모한다.
  - **운영체제 스케줄러 설정**: 새로운 스레드가 생성되면 운영체제의 스케줄러가 이를 관리해야 하므로, 추가적인 오버헤드가 발생한다.
- 이처럼 스레드 생성 비용이 크기 때문에, **가벼운 작업을 처리할 때마다 스레드를 새로 만들면 작업 시간보다 스레드 생성 시간이 더 오래 걸리는** 비효율이 발생할 수 있다.
- 이 문제를 해결하려면, **생성한 스레드를 재사용**하는 방법을 고려해야 한다.

### 1.2. 스레드 관리 문제

- 시스템의 CPU나 메모리 자원은 한정되어 있으므로, **스레드를 무한정 만들 수는 없다**.
- 만약 사용자의 요청이 들어올 때마다 스레드를 계속 생성하면, 사용자가 몰릴 때 시스템 자원이 고갈되어 서버가 다운될 수 있다. 따라서 시스템이 감당할 수 있는 **최대 스레드 개수를 관리**해야 한다.
- 또한, 애플리케이션을 안전하게 종료하려면 실행 중인 스레드들이 **남은 작업을 마치도록 기다리거나, 급하게 종료해야 할 때 인터럽트 신호**를 보내야 한다. 이러한 작업을 위해서도 스레드는 어딘가에서 관리되어야 한다.

### 1.3. Runnable 인터페이스의 불편함

- **`Runnable` 인터페이스**는 스레드 작업을 정의하는 데 사용되지만, 몇 가지 불편함이 있다.
  - **반환 값 부재**: `run()` 메서드는 반환 값이 없으므로, **스레드의 실행 결과를 직접 받을 수 없다**. 결과를 얻으려면 멤버 변수에 값을 저장한 뒤, `join()` 등으로 스레드가 종료되기를 기다려야 하는 번거로움이 있다.
  - **예외 처리의 제약**: `run()` 메서드는 **체크 예외(Checked Exception)를 밖으로 던질 수 없다**. 따라서 예외 처리는 반드시 `run()` 메서드 내부에서 `try-catch`로 처리해야 한다.

### 1.4. 해결 방법: 스레드 풀과 Executor 프레임워크

- 앞서 언급한 문제들을 해결하기 위해 **스레드 풀(Thread Pool)** 이라는 개념이 필요하다.
- 스레드 풀은 **필요한 만큼의 스레드를 미리 만들어두고, 재사용**하는 방식이다.
  1.  작업 요청이 오면, 스레드 풀에서 대기 중인 스레드를 하나 꺼내 작업을 처리한다.
  2.  작업이 완료되면, 스레드를 종료하는 대신 **다시 스레드 풀에 반납하여 재사용**을 준비한다.
- 스레드 풀을 사용하면 **스레드 생성 비용을 절약**할 수 있고, **최대 스레드 개수를 관리**하여 시스템을 안정적으로 유지할 수 있다.
- 이러한 스레드 풀의 복잡한 동작(생산자-소비자 패턴, 스레드 상태 관리 등)을 직접 구현하는 것은 매우 어렵다.
- 자바는 이 모든 문제를 한 번에 해결해주는 **Executor 프레임워크**를 제공한다.

## 2. Executor 프레임워크

- **Executor 프레임워크**는 멀티스레딩 및 병렬 처리를 쉽게 사용할 수 있도록 돕는 **기능 모음**이다.
- 작업의 실행과 스레드 풀을 효율적으로 관리하여, 개발자가 **직접 스레드를 생성하고 관리하는 복잡함을 줄여준다**.

### 2.1. 주요 구성 요소

- Executor 프레임워크는 여러 인터페이스와 클래스로 구성되며, 그 중심에는 `Executor`와 `ExecutorService` 인터페이스가 있다.

#### Executor 인터페이스

```java
package java.util.concurrent;

public interface Executor {
    void execute(Runnable command);
}
```

- `Executor`는 가장 단순한 작업 실행 인터페이스로, **`execute(Runnable command)`** 메서드 하나만을 가진다.

#### ExecutorService 인터페이스

```java
package java.util.concurrent;

public interface ExecutorService extends Executor, AutoCloseable {
    <T> Future<T> submit(Callable<T> task);

    @Override
    default void close() {
        //...
    };
    // ...
}
```

- `ExecutorService`는 `Executor` 인터페이스를 확장하여, **작업의 제출 및 제어 기능을 추가로 제공**하는 핵심 인터페이스이다.
- 주요 메서드로는 `submit()`과 `close()` 가 있다.
- Executor 프레임워크를 사용할 때는 **대부분 이 인터페이스를 사용**하며, 주요 구현체로는 `ThreadPoolExecutor`가 있다.

## 3. TheadPoolExecutor

- `ExecutorService`의 핵심 구현체로, 크게 **스레드 풀(Thread Pool)** 과 **작업 큐(Task Queue)**, 두 가지 요소로 구성된다.
- **스레드 풀**은 스레드를 생성하고 관리하며, **작업 큐**는 제출된 작업들을 보관하는 역할을 한다. 이때 작업 큐는 생산자-소비자 문제를 해결하기 위해 일반 큐가 아닌 `BlockingQueue`를 사용한다.
- `executor.execute(new Task())`와 같이 작업을 제출하면, 해당 작업(`Task`)은 `BlockingQueue`에 보관된다. 이 과정에서 작업을 제출하는 스레드(예: `main` 스레드)가 **생산자**가 되고, 스레드 풀에 있는 스레드들이 **소비자**가 되어 큐에 보관된 작업을 가져가 처리하는 **생산자-소비자 패턴**으로 동작한다.

### 3.1. ThreadPollExecutor 생성자

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) { ... }
```

- `corePoolSize`: **핵심 스레드 수**. 스레드 풀에서 스레드의 수이다.
- `maximumPoolSize`: **최대 스레드 수**. 스레드 풀에서 관리하는 최대 스레드의 수이다.
- `keepAliveTime`, `unit`: **최대 스레드의 유휴 시간**. `maximumPoolSize`에 의해 생성된 스레드가 이 시간 동안 아무 작업도 하지 않으면 제거된다.
- `workQueue`: **작업 큐**. 제출된 `Runnable` 작업을 보관하는 `BlockingQueue`이다.

## 4. Future

### 4.1. Runnable과 Callable 비교

#### Runnable

```java
package java.lang;

public interface Runnable {
    void run();
}
```

- `run()` 메서드의 반환 타입이 `void`이므로, **작업의 결과를 반환할 수 없다**.
- 메서드 시그니처에 예외가 선언되어 있지 않으므로, **체크 예외(Checked Exception)를 밖으로 던질 수 없다**.

#### Callable

```java
package java.util.concurrent;

public interface Callable<V> {
    V call() throws Exception;
}
```

- `call()` 메서드의 반환 타입이 제네릭 `V`이므로, **작업의 결과를 반환할 수 있다**.
- `throws Exception`이 선언되어 있어, **체크 예외를 밖으로 던질 수 있다**.

#### Callable과 Future 사용

- `ExecutorService`의 `submit(Callable<T> task)` 메서드를 통해 `Callable` 작업을 전달할 수 있다.
- 작업을 제출하면 `ExecutorService`는 즉시 **`Future`** 라는 특별한 객체를 반환한다.
- 이후, `future.get()` 메서드를 호출하면, **`Callable`의 `call()` 메서드가 반환한 실제 결과를 받아올 수 있다**.

```java
// Callable 작업을 제출하고 즉시 Future를 받는다.
Future<Integer> future = executorService.submit(new MyCallable());

// ... 다른 작업을 수행 ...

// future.get()을 호출하여 결과를 받는다.
Integer result = future.get();
```

- 이처럼 `Callable`과 `Future`를 사용하면, `Runnable`의 한계였던 **반환 값과 예외 처리를 매우 편리하게 해결**할 수 있다.
