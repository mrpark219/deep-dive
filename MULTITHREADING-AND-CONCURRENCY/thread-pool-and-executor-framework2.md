# 스레드 풀과 Executor 프레임워크2

## 1. 우아한 종료

- 만약 고객의 주문을 처리하는 서버를 기능 업데이트를 위해 재시작해야 한다고 가정한다.
- 이때, **고객의 주문이 처리되는 도중에 서버가 갑자기 재시작**된다면 해당 주문은 제대로 완료되지 못할 것이다.
- 가장 이상적인 방식은 **새로운 주문은 더 이상 받지 않고, 이미 진행 중인 주문은 모두 완료시킨 뒤에 서버를 재시작**하는 것이다.
- 이처럼 실행 중인 작업을 안전하게 마무리하고 시스템을 종료하는 방식을 **우아한 종료(Graceful Shutdown)** 라고 한다.

## 2. ExecutorService의 종료 메서드

- `ExecutorService`는 우아한 종료를 위해 여러 가지 종료 관련 메서드를 제공한다.

### 2.1. 서비스 종료

- void shutdown()
  - **새로운 작업을 더 이상 받지 않고**, 작업 큐에 **이미 제출된 작업들이 모두 완료된 후에** 스레드 풀을 종료시킨다.
  - **논블로킹(Non-blocking)** 메서드로, 호출한 스레드는 즉시 다음 코드를 실행한다.
- List<Runnable> shutdownNow()
  - **즉시 스레드 풀을 종료**한다.
  - **실행 중인 작업에게는 인터럽트(interrupt) 신호**를 보내 중단을 시도하고, **작업 큐에서 대기 중이던 작업들의 목록을 반환**한다.
  - **논블로킹** 메서드이다.

### 2.2. 서비스 상태 확인

- boolean isShutdown()
  - `shutdown()`이나 `shutdownNow()`가 호출되어 **종료 절차가 시작되었는지** 여부를 확인한다.
- boolean isTerminated()
  - 종료 절차 시작 후, **모든 작업이 실제로 완료되어 스레드 풀이 완전히 종료되었는지** 여부를 확인한다.

### 2.3. 작업 완료 대기

- boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException
  - `shutdown()` 호출 이후, **모든 작업이 완료될 때까지 지정된 시간 동안 대기**한다.
  - **블로킹(blocking)** 메서드이다.
  - 지정된 시간 내에 모든 작업이 완료되면 `true`를, 시간이 초과되어도 완료되지 않으면 `false`를 반환한다.

### 2.4. close()

- 자바 19부터 지원하는 메서드로, **`shutdown()`과 동일하게 동작**한다.
- 더 정확하게는, `shutdown()`을 호출하고 **하루를 기다려도 작업이 완료되지 않으면 `shutdownNow()`를 호출**한다.
- 또한, `close()`를 호출한 스레드에 인터럽트가 발생해도 `shutdownNow()`를 호출한다.

## 3. Executor 스레드 풀 관리

- `ExecutorService`의 기본 구현체인 `ThreadPoolExecutor`의 생성자는 다음 핵심 속성들을 사용하여 스레드 풀을 설정한다.
- corePoolSize
  - 스레드 풀에서 **기본적으로 유지**하는 스레드의 수이다.
- maximumPoolSize
  - 스레드 풀이 **최대로 확장**할 수 있는 스레드의 수이다.
- keepAliveTime, unit
  - **`corePoolSize`를 초과**하여 생성된 스레드(초과 스레드)가 **새로운 작업을 기다리는 최대 생존 시간**이다. 이 시간 동안 처리할 작업이 없으면 해당 스레드는 제거된다.
- workQueue
  - 스레드가 처리할 **작업을 보관**하는 `BlockingQueue`이다.

### 3.1. ThreadPoolExecutor 생성 예시

```java
BlockingQueue<Runnable> workQueue = new ArrayBlockingQueue<>(2);
ExecutorService es = new ThreadPoolExecutor(2, 4, 3000, TimeUnit.MILLISECONDS, workQueue);
```

- new ArrayBlockingQueue<>(2)
  - 작업을 보관할 **블로킹 큐**로 `ArrayBlockingQueue`를 사용하며, 최대 2개까지 작업을 큐에 보관할 수 있다.
- corePoolSize = 2
  - 스레드 풀은 기본적으로 **2개의 스레드(핵심 스레드)** 를 유지한다.
- maximumPoolSize = 4
  - 작업 요청이 많아져 **작업 큐(2개)가 가득 차면**, 스레드 풀은 **최대 4개까지 스레드를 추가로 생성**하여 작업을 처리한다.
- 3000, TimeUnit.MILLISECONDS
  - `corePoolSize`를 초과하여 생성된 **초과 스레드**가 3000ms(3초) 동안 아무 작업도 하지 않으면, 해당 스레드는 **제거**된다.

### 3.2. 스레드 미리 생성하기

- 응답 시간이 아주 중요한 서버라면, 고객의 **첫 요청을 받기 전에 스레드를 스레드 풀에 미리 생성**해두고 싶을 수 있다.
- 스레드를 미리 생성해두면, 첫 요청을 처리하는 스레드의 생성 시간을 줄여 **응답 속도를 향상**시킬 수 있다.
- `ThreadPoolExecutor`는 **`prestartAllCoreThreads()`** 메서드를 제공하며, 이 메서드를 호출하면 **핵심 스레드(core threads)를 미리 생성**해 둘 수 있다.
- 이 메서드는 `ThreadPoolExecutor` 구현 클래스에만 존재하며, 상위 인터페이스인 `ExecutorService`는 이 메서드를 제공하지 않는다.

## 4. Executor 전략

- `ThreadPoolExecutor`를 사용하면 스레드 풀의 스레드 수와 블로킹 큐 등 다양한 속성을 조절할 수 있다.
- 자바는 `Executors` 클래스를 통해 3가지 기본 **전략**을 제공한다.
  - `newSingleThreadExecutor()`: **단일 스레드 풀** 전략이다.
  - `newFixedThreadPool(nThreads)`: **고정 스레드 풀** 전략이다.
  - `newCachedThreadPool()`: **캐시 스레드 풀** 전략이다.

### 4.1. 단일 스레드 풀 전략: newSingleThreadExecutor()

- 스레드 풀에 **핵심 스레드(corePoolSize) 1개**만 사용하는 전략이다.
- 큐 사이즈에 제한이 없는 **`LinkedBlockingQueue`** 를 사용한다.
- 주로 간단한 작업이나 테스트 용도로 사용한다.

```java
new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
```

### 4.2. 고정 스레드 풀 전략: newFixedThreadPool(nThreads)

- 스레드 풀에 **`nThreads` 만큼의 기본 스레드**를 생성하며, **최대 스레드 수(`maximumPoolSize`)도 이와 동일하게 설정**한다. 즉, 초과 스레드를 생성하지 않는다.
- 작업 큐는 `LinkedBlockingQueue`를 사용하여 사실상 큐 사이즈에 제한이 없다.
- 스레드 수가 **고정**되어 있기 때문에, **CPU와 메모리 사용량을 예측**할 수 있어 안정적인 운영이 가능하다.
- 하지만, **요청이 처리되는 속도보다 쌓이는 속도가 더 빠르면** 심각한 문제가 발생할 수 있다. 큐 사이즈에 제한이 없으므로, **작업이 `LinkedBlockingQueue`에 수만 건씩 계속 쌓이게** 된다.
- 이 경우, 서버의 **CPU나 메모리 자원은 여유가 있어 보이지만**, 사용자들은 **응답을 받기까지 점점 더 오래 기다려야 하는** 문제가 발생한다.

```java
new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
```

### 4.3. 캐시 풀 전략: newCachedThreadPool()

- 이 전략은 **핵심 스레드(`corePoolSize`)를 0**으로 설정하고, **`maximumPoolSize`에는 사실상 제한(Integer.MAX_VALUE)을 두지 않는** 방식이다.
- 작업 큐(workQueue)로는 `SynchronousQueue`를 사용하여, 작업을 전혀 저장하지 않고 **생산자 스레드가 소비자 스레드에게 직접 작업을 전달**하게 한다.
- `keepAliveTime`은 60초로 설정되어, 작업이 없는 스레드는 60초 후 제거된다.
- 작업 요청이 오면 `SynchronousQueue`는 작업을 저장하는 대신, **즉시 스레드 풀에 스레드가 있는지 확인**한다.
  - 만약 쉬고 있는 스레드가 있다면 그 스레드에게 작업을 전달한다.
  - 만약 쉬고 있는 스레드가 없다면 **즉시 새로운 스레드를 생성**하여 작업을 처리한다.
- 모든 요청이 큐에서 대기하지 않고 스레드에 의해 **바로바로 처리**되므로, **매우 빠른 응답 속도**를 기대할 수 있다.
- 하지만 **초과 스레드 수에 제한이 없기** 때문에, 요청이 급증할 경우 스레드가 무한정 생성되어 **시스템의 CPU와 메모리 자원을 모두 소모**시켜 장애가 발생할 수 있다.

#### SynchronousQueue

- **내부에 저장 공간이 없는(크기가 0)** 특별한 `BlockingQueue`이다.
- 이는 생산자 스레드가 `put()`을 호출하면, 소비자 스레드가 `take()`를 호출할 때까지 **생산자 스레드가 블로킹(blocking)** 되도록 강제한다. (그 반대의 경우도 마찬가지이다.)
- 즉, 중간 버퍼 없이 **생산자와 소비자가 1:1로 직접 데이터를 전달(hand-off)하도록 동기화**하는 큐이다.

```java
new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
```

### 4.4. 사용자 정의 풀 전략

- **고정 스레드 풀** 전략은 서버 자원은 여유가 있는데 사용자가 대기하는 문제가 발생할 수 있고, **캐시 스레드 풀** 전략은 서버 자원을 최대로 사용하지만 시스템 다운의 위험이 있다.
- 따라서 일반적인 상황에서는 **고정 크기의 스레드**로 서비스를 안정적으로 운영하고, 요청이 급증하는 **긴급 상황에는 스레드를 추가로 투입**하여 작업을 빠르게 처리하며, 요청이 폭증하여 감당이 안 될 때는 **새로운 요청을 거절**하는 **사용자 정의 전략**이 가장 이상적이다.
- 예를 들어, 아래와 같이 세분화된 전략을 사용할 수 있다.
  - **핵심 스레드(`corePoolSize`) 100개**를 기본으로 사용한다.
  - 긴급 대응을 위해 **최대 스레드(`maximumPoolSize`)를 200개**로 설정하여, 100개의 스레드를 추가로 투입할 수 있게 한다.
  - 추가된 스레드는 **60초의 생존 시간(`keepAliveTime`)**을 가지며, 60초간 작업이 없으면 제거된다.
  - 작업 큐(`workQueue`)는 **1000개의 용량**을 가지며, 1000개를 초과하는 요청은 거절된다.

```java
new ThreadPoolExecutor(100, 200, 60L, TimeUnit.SECONDS, new ArrayBlockingQueue<>(1000));
```
