# 자바 메모리 구조

## 1. 자바 메모리 구조

### 1.1. 메서드 영역 (Method Area)

- **메서드 영역**은 프로그램을 실행하는 데 필요한 **공통 데이터를 관리**하는 곳으로, **모든 스레드가 공유**하는 영역이다.
- 이 영역에는 주로 다음과 같은 정보가 저장된다.
  - **클래스 정보**: 클래스의 바이트 코드, 필드, 메서드, 생성자 등 모든 실행 코드가 포함된다.
  - **static 영역**: `static` 키워드로 선언된 변수들이 보관된다.
  - **런타임 상수 풀(Runtime Constant Pool)**: 프로그램 실행에 필요한 리터럴 상수들이 보관된다.

### 1.2. 스택 영역 (Stack Area)

- **스택 영역**은 **스레드마다 개별적으로 생성**되는 메모리 영역이다. 즉, 스레드의 개수만큼 스택 영역이 존재한다.
- 스택 영역에는 **스택 프레임(Stack Frame)** 이라는 구조가 쌓인다. **메서드가 호출될 때마다 하나의 스택 프레임이 생성**되어 스택에 추가되고, **메서드가 종료되면 해당 프레임은 스택에서 제거**된다.
- 각 스택 프레임에는 해당 메서드의 **지역 변수, 중간 연산 결과, 다른 메서드 호출 정보** 등이 포함된다.

### 1.3. 힙 영역 (Heap Area)

- **힙 영역**은 `new` 키워드를 통해 생성된 **객체(인스턴스)와 배열이 저장**되는 공간이다.
- 이 영역은 **모든 스레드가 공유**하며, **가비지 컬렉션(GC)** 이 발생하는 주요 대상이다.
- 더 이상 어떤 곳에서도 참조되지 않는 객체는 **GC에 의해 자동으로 메모리에서 제거**된다.

## 2. 스레드 생성 방법

- 자바에서 스레드를 생성하는 방법은 크게 **`Thread` 클래스를 상속**받는 방법과 **`Runnable` 인터페이스를 구현**하는 방법, 두 가지가 있다.

### 2.1. Thread 클래스 상속

- 자바에서는 스레드 역시 객체로 다루므로, **`Thread` 클래스를 상속**받고 **`run()` 메서드를 재정의**하여 스레드가 실행할 코드를 작성한다.
- `Thread.currentThread().getName()`을 호출하면 현재 코드를 실행 중인 스레드의 이름을 조회할 수 있다.

```java
public class HelloThread extends Thread {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ": run()");
    }
}
```

```java
public static void main(String[] args) {
    System.out.println(Thread.currentThread().getName() + ": main() start");
    HelloThread helloThread = new HelloThread();
    helloThread.start();
    System.out.println(Thread.currentThread().getName() + ": main() end");
}
```

- 자바 프로그램이 실행되면, JVM은 **`main`이라는 이름의 기본 스레드를 생성**하고 `main()` 메서드를 실행한다.
- `main` 스레드에서 `helloThread.start()`를 호출하면, JVM은 **새로운 스레드(기본 이름: `Thread-0`)를 위한 별도의 스택 공간을 할당**하고, `run()` 메서드를 실행시킨다.
- 이때, `main` 스레드와 `Thread-0` 스레드는 **독립적으로 실행**된다. **운영체제의 스케줄링에 따라 어떤 스레드가 먼저 실행될지, 얼마나 오래 실행될지는 보장되지 않는다**.

#### `start()` 대신 `run()`을 직접 호출하면?

```java
HelloThread helloThread = new HelloThread();
helloThread.run(); // start()가 아닌 run() 직접 호출
```

- `start()` 대신 `run()` 메서드를 직접 호출하면, **새로운 스레드가 생성되지 않는다**.
- 이는 단순히 **`main` 스레드가 `HelloThread` 객체의 `run()`이라는 메서드를 호출**하는 것과 같다.
- 따라서 `run()` 메서드의 스택 프레임은 새로운 스레드의 스택이 아닌, **`main` 스레드의 스택 위에 쌓이게** 된다.
- 결론적으로, 멀티스레딩으로 동작하게 하려면, 즉 **별도의 스레드에서 `run()` 메서드를 실행하려면 반드시 `start()` 메서드를 호출**해야 한다.
