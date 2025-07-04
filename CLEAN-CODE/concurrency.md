# 동시성

- **동시성과 깔끔한 코드는 양립하기 어렵다**.
- 스레드를 하나만 실행하는 코드는 구현이 단순하다.
- 하지만 멀쩡해 보이지만 내부에 결함이 숨겨진 다중 스레드 코드를 작성하기 쉽다.
- 이러한 코드는 시스템에 부하가 걸리기 전까지는 문제 없이 동작하는 것처럼 보인다.

## 1. 동시성이 필요한 이유?

- **동시성은 결합(coupling)을 없애는 전략이다**.
- 이는 **무엇(what)** 과 **언제(when)** 를 분리하는 방식이다.
- 스레드가 하나뿐인 프로그램에서는 무엇과 언제가 서로 밀접하게 연관되어 있다.
- 따라서 호출 스택만 확인해도 프로그램 상태를 쉽게 파악할 수 있다.
- 단일 스레드 프로그램은 디버깅 시 정지점(breakpoint)을 설정해 현재 상태를 추적할 수 있다.
- **무엇과 언제를 분리하면 애플리케이션 구조와 효율이 극적으로 향상된다**.
- 구조적 측면에서 보면 프로그램은 하나의 거대한 루프가 아니라 **작은 협력 프로그램들로 구성된다**.
- 이로 인해 시스템 이해도와 문제 분리도 용이해진다.
- 일부 시스템은 **응답 시간과 작업 처리량(throughput)** 향상 요구 때문에 **직접적인 동시성 구현이 필수적이다**.

### 1.1 미신과 오해

- 반드시 동시성을 도입해야 하는 상황이 존재하지만, **동시성은 어렵고 신중해야 한다**.

#### 잘못된 믿음

- **동시성은 항상 성능을 높여준다**.
  - 동시성은 때로 성능을 높일 수 있지만, 이는 다음 조건에서만 해당한다.
    - 대기 시간이 길어 **여러 스레드가 프로세서를 공유할 수 있는 경우**
    - 여러 프로세스가 **동시에 독립적인 계산을 수행할 수 있는 경우**
    - **이러한 조건은 흔치 않다**.
- **동시성을 구현해도 설계는 바뀌지 않는다**.
  - 단일 스레드 시스템과 다중 스레드 시스템은 설계가 완전히 다르다.
  - 무엇과 언제를 분리하면 시스템 구조 자체가 **크게 달라진다**.
- **웹 또는 EJB 컨테이너를 사용하면 동시성을 이해하지 않아도 된다**.
  - 실제로는 컨테이너 내부 동작 방식과, **동시 수정이나 데드락을 피하는 방법**을 반드시 이해해야 한다.

#### 타당한 사실

- **동시성은 부하를 유발한다**.
  - 성능에 영향을 줄 수 있으며, **더 많은 코드를 작성해야 한다**.
- **동시성은 복잡하다**.
  - 간단한 문제조차도 동시성을 적용하면 **복잡도가 급격히 증가한다**.
- **동시성 버그는 재현하기 어렵다**.
  - 그래서 흔히 **일회성 문제로 오인되어 무시되기 쉽다**.
- **동시성을 구현하려면 근본적인 설계 전략을 다시 생각해야 한다**.

## 2. 난관

```java
public class X {
    private int lastIdUsed;

    public int getNextId() {
        return ++lastIdUsed;
    }
}
```

- 위 코드는 인스턴스 `X`를 생성하고 `lastIdUsed` 필드를 42로 설정한 다음, 두 스레드가 해당 인스턴스를 공유하는 상황을 가정한다.
- 이 상태에서 두 스레드가 `getNextId()`를 호출할 경우 다음과 같은 **세 가지 결과 중 하나가 발생할 수 있다.**

  1. **한 스레드는 43을 받고, 다른 스레드는 44를 받으며, `lastIdUsed`는 44가 된다.**
  2. **한 스레드는 44를 받고, 다른 스레드는 43을 받으며, `lastIdUsed`는 44가 된다.**
  3. **두 스레드가 모두 43을 받고, `lastIdUsed`는 43이 된다.**

- 세 번째 경우는 **명백한 오류**이다.
- 이는 두 스레드가 같은 변수에 **동시에 접근**하면서 발생한다.
- 자바 코드 한 줄을 **여러 스레드가 동시에 실행**할 경우, **내부적으로 수많은 실행 경로**가 존재하고, 그 중 일부는 **잘못된 결과**를 만들어낸다.

## 3. 동시성 방어 원칙

### 3.1 단일 책임 원칙(Single Responsibility Principle, SRP)

- SRP는 주어진 메서드, 클래스, 컴포넌트를 변경할 이유가 하나여야 한다는 원칙이다.
- 동시성은 복잡성 하나만으로도 따로 분리할 이유가 충분하다.
- **동시성 관련 코드는 다른 코드와 분리해야 한다**.
- 동시성 코드는 독자적인 개발, 변경, 조율 주기가 있다.
- **동시성 문제는 다른 코드와 다른 종류의 난관을 수반하며, 더 어렵다**.
- 잘못 구현한 동시성 코드는 별의별 방식으로 실패한다.

### 3.2 따름 정리(corollary): 자료 범위를 제한하라

- **공유 객체의 동일 필드를 두 스레드가 수정하면 예기치 않은 결과가 발생할 수 있다**.
- 이를 방지하기 위해 **공유 객체에 접근하는 코드를 `synchronized`로 감싸는 방식이 일반적이다**.
- **하지만 임계 영역의 수를 줄이는 것이 핵심이다**.
- 공유 자료를 줄이고, 캡슐화하며, 가능하면 독립적인 단위로 분할해야 한다.
- 보호해야 할 임계 영역을 빠뜨리면 공유 자료를 수정하는 모든 코드가 깨질 수 있다.
- 모든 임계 영역이 제대로 보호되었는지 확인하려면 많은 수고가 든다.
- 동시성 버그는 **발견이 어렵고**, 문제를 파악하는 데 시간이 오래 걸릴 수 있다.

#### 따름 정리: 자료 사본을 사용하라

- **처음부터 공유하지 않는 것이 가장 좋은 전략이다**.
- 경우에 따라 객체를 복사해 읽기 전용으로 사용하거나, 각 스레드가 사본을 만들어 결과만 가져오게 할 수 있다.
- 복사에 시간이 들 수 있지만, **동기화를 피한다면 오히려 성능이 좋아질 수 있다**.

#### 따름 정리: 스레드는 가능한 독립적으로 구현하라

- **다른 스레드와 자료를 공유하지 않는 스레드를 구현하면 복잡도를 줄일 수 있다**.
- 각 스레드는 클라이언트 요청 하나만 처리하고, 필요한 정보는 로컬 변수에 저장한다.
- 이렇게 하면 동기화 없이 독립적으로 동작할 수 있다.

## 4. 라이브러리를 이해하라

- **언어나 프레임워크에서 제공하는 동시성 라이브러리를 잘 이해하고 활용해야 한다**.
- 동기화, 락, 스레드 풀, 큐 등은 대부분의 언어에서 표준 라이브러리로 제공된다.
- **직접 구현하기보다 검증된 라이브러리를 사용하는 것이 안정성과 생산성을 높이는 길이다**.
- 예를 들어 Java에서는 `synchronized`, `volatile`, `ReentrantLock`, `ExecutorService`, `ConcurrentHashMap` 등이 있다.

## 5. 실행 모델을 이해하라

### 5.1 기본 용어

- 한정된 자원(Bound Resource): 다중 스레드 환경에서 사용하는 자원이며, **크기나 숫자가 제한적**이다. 데이터베이스 연결, 길이가 일정한 읽기/쓰기 버터 등이 있다.
- 상호 배제(Mutual Exclusion)는 **한 번에 한 스레드만 공유 자원에 접근할 수 있는 상태**이다.
- 기아(Starvation)는 일부 스레드가 오랫동안 자원을 기다리는 상태이다.
- 데드락(Deadlock)은 **스레드들이 서로 자원을 점유한 채 상대방을 기다려 진행되지 못하는 상태**이다.
- 라이브락(Livelock)은 **스레드들이 서로 양보하느라 계속 상태를 바꾸지만 진전이 없는 상태**이다.

### 5.2 실행 모델: 생산자-소비자(Producer-Consumer)

- 이 모델은 하나 이상의 생산자 스레드가 데이터를 큐(버퍼)에 넣고, 소비자 스레드가 이를 꺼내 처리하는 구조이다.
- 큐는 한정된 자원이므로 시그널 타이밍이 어긋나면 **양쪽 스레드가 서로의 시그널을 기다리는 교착 상태가 발생할 수 있다**.

**💡 해결 전략**

- `BlockingQueue` 같은 **스레드 안전한 큐를 사용**한다.
- `notify()` 대신 `notifyAll()`을 사용해 모든 대기 스레드에 알림을 전파한다.

### 5.3 실행 모델: 읽기-쓰기(Readers-Writers)

- 이 모델은 읽기 스레드는 자원을 읽고, 쓰기 스레드는 자원을 갱신하는 구조이다.
- 읽기 성능을 높이려다 보면 **쓰기 스레드가 기아 상태에 빠지거나 정보가 오래된 상태로 유지될 수 있다**.

**💡 해결 전략**

- `ReadWriteLock`을 사용해 **다수의 읽기는 동시에 허용하고**, **쓰기는 단독 접근을 보장**한다.
- `ReentrantReadWriteLock(true)`를 사용해 공정성(fairness)을 고려한다.

### 5.4 실행 모델: 식사하는 철학자들 (Dining Philosophers)

- 이 모델은 **여러 스레드가 동시에 자원을 공유하면서도 데드락 없이 작동할 수 있는지를 설명하는 고전적인 문제**이다.
- N명의 철학자가 **원형 테이블에 앉아 있으며**, 각 철학자 사이에는 **하나의 포크(자원)** 가 놓여 있다.
- 각 철학자는 **생각과 식사를 번갈아 반복**한다.
- 식사를 하려면 **왼쪽과 오른쪽 포크를 모두 들어야 하며**, 그렇지 않으면 식사를 할 수 없다.

#### ⚠️ 발생 가능한 문제

- 모든 철학자가 **동시에 왼쪽 포크만 들고 오른쪽 포크를 기다린다면**, 아무도 식사를 하지 못하게 되어 **데드락(Deadlock)** 이 발생한다.
- 또는 포크를 서로 양보하며 **끊임없이 시도만 하고 식사를 하지 못하는 라이브락(Livelock)** 상태에 빠질 수 있다.
- 어떤 철학자는 계속 기회를 얻지 못해 **기아(Starvation)** 상태에 놓일 수도 있다.

#### 💡 해결 전략

- **N-1명만 식탁에 동시에 앉게 제한한다**

  - 예: 5명의 철학자가 있다면 **최대 4명만 식사 시도 가능**하게 한다.
  - 최소 한 명은 항상 양쪽 포크를 들 수 있어 **데드락을 방지할 수 있다**.

- **두 포크를 동시에 획득하도록 시도하고**, 하나라도 실패하면 **모두 내려놓고 다시 시도한다**

  - `tryLock()`을 활용해 포크를 얻을 수 없을 때 **기다리지 않고 빠르게 포기**하게 한다.
  - 이 방식은 **데드락과 라이브락을 효과적으로 방지**한다.

- **포크를 드는 순서를 철학자마다 다르게 지정한다**
  - **짝수 번호 철학자는 왼쪽 포크를 먼저**, **홀수 번호 철학자는 오른쪽 포크를 먼저 들도록 한다**.
  - 이를 통해 **순환 대기 조건을 깨뜨려 데드락 가능성을 줄인다**.

## 6. 동기화 메서드 간의 의존성을 이해하라

- 동기화된 메서드 사이에 **의존성이 존재하면 예측하기 어려운 버그가 생긴다**.
- 자바에서는 `synchronized` 키워드로 개별 메서드를 보호할 수 있지만, 공유 클래스에 동기화된 메서드가 여럿 존재하면 구현이 올바른지 반드시 확인해야 한다.
- **공유 객체에는 가급적 메서드 하나만 사용하는 것이 바람직하다**.

공유 객체에 여러 메서드가 필요한 경우 다음 방법을 고려한다.

- **클라이언트에서 잠금**: 클라이언트가 첫 번째 메서드를 호출하기 전에 서버 객체를 잠그고, 마지막 메서드까지 호출한 후 잠금을 해제한다.
- **서버에서 잠금**: 서버에 모든 메서드를 포함하는 통합 메서드를 구현하고, 이 내부에서 잠금을 수행하도록 한다.
- **연결(Adapted) 서버 구성**: 잠금을 수행하는 중간 단계를 생성해 동기화를 처리하고, 기존 서버는 그대로 유지한다.

## 7. 동기화하는 부분을 작게 만들어라

- 자바에서 `synchronized` 키워드를 사용하면 락을 설정한다. 같은 락으로 감싼 모든 코드 영역은 한 번에 한 스레드만 실행할 수 있다.
- 락은 스레드를 지연시키고 부하를 가중시킨다. 따라서 `synchronized` 문은 **필요한 최소 범위에서만 사용해야 한다**.
- 반면, **임계영역(critical section)은 반드시 보호해야 한다**.
- **임계영역의 수를 줄이는 것이 목표이나**, 이를 위해 **오히려 임계영역을 지나치게 키우면 문제가 된다**.
- 필요 이상으로 임계영역 크기를 키우면 **스레드 간 경쟁이 늘어나고, 프로그램 성능이 떨어진다**.

## 8. 올바른 종료 코드는 구현하기 어렵다

- **영구적으로 동작하는 시스템과 일시적으로 작동하는 시스템은 종료 처리 방식이 다르다**.
- **다중 스레드 환경에서 깔끔하게 종료하는 코드를 구현하기는 매우 어렵다**.
- 가장 흔한 문제는 **데드락이다**. 스레드가 **도착하지 않을 시그널을 기다리며 멈춘다**.
- 예를 들어 부모 스레드가 여러 자식 스레드를 생성하고, **모두 종료될 때까지 기다렸다가 자원을 해제하고 종료하는 구조가 있다**고 하자.
  - 이때 자식 스레드 중 하나가 데드락에 걸리면, **부모 스레드는 영원히 종료되지 못한다**.
- **종료 처리가 필요한 다중 스레드 코드를 작성할 때는, 시간을 들여 제대로 설계하고 구현해야 한다**.

## 9. 스레드 코드 테스트하기

- 문제를 노출하는 테스트 케이스를 작성해야 한다.
- 프로그램 설정, 시스템 설정, 부하 조건을 다양하게 바꿔가며 테스트해야 한다.
- 테스트가 실패하면 반드시 원인을 추적해야 한다. **한두 번 실패한 후 통과한다고 넘어가서는 안 된다.**

### 9.1 말이 안되는 실패는 잠정적인 스레드 문제로 취급하라

- **다중 스레드 코드는 간헐적이고 설명되지 않는 오류를 일으킬 수 있다.**
- 스레드 버그는 수천, 수백만 번 중에 한 번 드러날 수 있다.
- 일회성 문제라고 판단하면 문제는 계속 숨어 있게 된다. **모든 실패는 원인을 찾아야 한다.**

### 9.2 다중 스레드를 고려하지 않은 순차 코드부터 제대로 돌게 만들자

- **먼저 스레드 환경 밖에서 코드가 정상 작동하는지 확인해야 한다.**
- 가능한 많은 코드를 POJO로 구성해 테스트 가능성을 높인다.
- **스레드와 무관한 버그와 스레드 관련 버그를 동시에 디버깅하지 않도록 한다.**

### 9.3 다중 스레드 코드를 다양한 환경에서 테스트할 수 있게 설계하라

- **한 스레드, 여러 스레드 환경 모두에서 코드가 잘 작동해야 한다.**
- **테스트 실행 속도도 다양하게 조절**하면서 반복 테스트를 수행한다.
- 테스트는 자동화하고, **빠르게/느리게/동시에 실행해 실패 가능성을 높인다.**

### 9.4 스레드 수를 조절할 수 있게 설계하라

- **적절한 스레드 수는 실험을 통해 결정해야 한다.**
- 스레드 수 조절이 쉽도록 유연하게 코드를 구성하고, **실행 중에도 조절이 가능하도록 구현한다.**

### 9.5 프로세서 수보다 많은 스레드를 돌려보라

- **스레드 스와핑이 발생하면, 임계영역을 빠뜨린 코드나 데드락이 드러날 가능성이 커진다.**
- 스와핑을 유도하기 위해 **프로세서 수보다 많은 스레드를 실행해야 한다.**

### 9.6 다른 플랫폼에서도 테스트하라

- **운영체제마다 스레드 스케줄링 정책이 다르므로, 예상치 못한 오류가 발생할 수 있다.**
- 처음부터 그리고 자주 **모든 목표 운영체제 환경에서 테스트를 수행해야 한다.**

### 9.7 코드에 보조 코드를 넣어 강제로 실패를 유도하라

- **스레드 관련 오류는 우발적이기 때문에 테스트로 발견되기 어렵다.**
- 보조 코드를 삽입하여 **코드 실행 순서를 인위적으로 바꾸면 오류 발생 가능성을 높일 수 있다.**
- 아래는 보조 코드를 추가하는 방법이다.

#### 보조 코드: 직접 구현하기

- `wait()`, `sleep()`, `yield()`, `priority()` 등의 함수를 코드에 삽입한다.
- **테스트 환경에서만 보조 코드를 실행하고, 배포 환경에서는 제거해야 한다.**
- 무작위성이 있기 때문에 오류가 반드시 발생하진 않는다.

#### 보조 코드: 자동화 도구 사용

- **AOF(Aspect Oriented Framework), CGLIB, ASM** 등을 활용하여 보조 코드를 자동으로 삽입할 수 있다.
- 코드를 인위적으로 흔들어 스레드 실행 순서를 매번 다르게 하여 오류를 유도한다.

## 10. 마무리

- **다중 스레드 코드는 올바르게 구현하기 어렵다.**
- **SRP 원칙을 지켜 스레드가 필요한 코드와 그렇지 않은 코드를 분리해야 한다.**
- **테스트 대상이 스레드 코드라면, 스레드 자체에 집중한 테스트를 수행해야 한다.**
- **동시성 오류가 발생하는 원인을 철저히 이해해야 한다.**
- 사용하는 **라이브러리, 알고리즘, 락 메커니즘**을 정확히 알아야 한다.
- **보호할 코드와 그 방법을 명확히 식별할 수 있어야 한다.**
- 문제는 언제든 생길 수 있다. **초기에는 일회성처럼 보이지만, 반드시 원인을 찾아야 한다.**
