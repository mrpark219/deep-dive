# 함수

## 1. 작게 만들어라

- 함수를 만드는 첫 번째 규칙은 **작게 만드는 것**이다.
- `if`, `else`, `while` 등의 블록은 **한 줄로 구성**하는 것이 바람직하다. 그 한 줄은 대부분 다른 함수를 호출하는 코드여야 한다.
- 블록 안에서 호출하는 함수의 **이름을 잘 지으면** 전체 흐름이 자연스럽고 이해하기 쉬워진다.
- 함수의 **들여쓰기 깊이는 1~2단계**를 넘지 않아야 한다.

## 2. 한 가지만 해라

- 함수는 **오직 한 가지 일만** 해야 한다. 그 일을 **잘** 해야 하며, **다른 일은 절대 하지 않아야** 한다.
- 함수 이름 아래에서 **동일한 추상화 수준의 작업만** 수행한다면 그 함수는 한 가지 일만 한다고 볼 수 있다.
- 함수를 분리하는 목적은 **큰 개념을 더 작은 추상화 수준으로 나누어 처리**하기 위해서다.
- 의미 있는 이름으로 함수를 **더 추출할 수 있다면**, 그 함수는 **여러 작업을 하고 있는 것**이다.

## 3. 내려가기 규칙

- 코드는 **위에서 아래로 읽히는 이야기처럼 자연스러워야** 한다.
- 한 함수 아래에는 **추상화 수준이 한 단계 낮은 함수**가 와야 한다.
- 이렇게 위에서 아래로 **단계적으로 세부화되는 구조**를 **내려가기 규칙**이라고 부른다.

## 4. Switch 문

- `switch` 문은 **여러 가지 처리를 포함**하므로 작게 만들기 어렵다.
- 본질적으로 **N가지 분기를 처리하는 구조**이며, 새로운 조건이 생기면 **코드를 변경**해야 한다.
- 따라서 `switch` 문은 **추상 팩토리로 감추고**, **적절한 파생 클래스 인스턴스를 생성**한 후 **공통 인터페이스를 통해 호출**하는 구조로 바꾸는 것이 좋다.

```java
// 1. 인터페이스 정의
public interface PaymentProcessor {
    void process();
}

// 2. 실제 구현체들 정의
public class KakaoPayProcessor implements PaymentProcessor {
    @Override
    public void process() {
        System.out.println("카카오페이 결제를 처리합니다.");
    }
}

public class NaverPayProcessor implements PaymentProcessor {
    @Override
    public void process() {
        System.out.println("네이버페이 결제를 처리합니다.");
    }
}

// 3. 추상 팩토리 클래스 정의
public abstract class PaymentProcessorFactory {
    public abstract PaymentProcessor createProcessor(String method);
}

// 4. 구현 팩토리 클래스
public class DefaultPaymentProcessorFactory extends PaymentProcessorFactory {
    @Override
    public PaymentProcessor createProcessor(String method) {
        return switch (method) {
            case "kakao" -> new KakaoPayProcessor();
            case "naver" -> new NaverPayProcessor();
            default -> throw new IllegalArgumentException("지원하지 않는 결제 방식: " + method);
        };
    }
}

// 5. 클라이언트 코드
public class PaymentService {
    private final PaymentProcessorFactory factory;

    public PaymentService(PaymentProcessorFactory factory) {
        this.factory = factory;
    }

    public void execute(String method) {
        PaymentProcessor processor = factory.createProcessor(method);
        processor.process();
    }
}

// 6. 실행
public class Main {
    public static void main(String[] args) {
        PaymentProcessorFactory factory = new DefaultPaymentProcessorFactory();
        PaymentService service = new PaymentService(factory);

        service.execute("kakao"); // 카카오페이 결제를 처리합니다.
        service.execute("naver");  // 네이버페이 결제를 처리합니다.
    }
}
```

## 5. 함수 인수

- 함수의 **이상적인 인수 개수는 0개**이다.
- 다음으로 좋은 수는 **1개**, 그 다음은 **2개**이며, **3개는 가급적 피해야** 한다.
- **4개 이상은 특별한 사유가 있어도 지양**해야 한다.
- 인수가 많아지면 **이해하기 어렵고, 테스트가 힘들며, 사용하는 입장에서도 실수할 가능성**이 높아진다.

### 5.1 단항 함수

- 인수 1개를 받는 대표적인 경우는 다음과 같다:
  - 어떤 조건을 확인하는 함수: `isDeleted(user)`
  - 변환을 수행하는 함수: `toEntity(dto)`
- 변환 함수에서 **출력 인수를 쓰는 것은 혼란을 유발**한다. **결과는 반환값으로 제공**해야 한다.
- **플래그 인수(boolean)** 는 절대 피해야 한다.  
  이는 **하나의 함수가 두 가지 일을 수행**한다는 뜻이며, 구조상 잘못된 설계다.

### 5.2 이항 함수

- 인수 2개인 함수는 **단항 함수보다 이해하기 어렵다**.
- 예를 들어 `write(String name, OutputStream out)` 함수는
  - `OutputStream`을 **클래스의 필드로 만들어 인수에서 제외**하거나
  - `FieldWriter` 같은 클래스를 만들어 **생성자에서 주입 후 `write(name)`처럼 단순화**하는 방식이 더 낫다.

### 5.3 삼항 함수

- 인수가 3개면 **이해와 사용이 더욱 어렵다**.
- 인수 순서 헷갈림, 파라미터 생략, 의미 추측 등 **문제 발생 가능성이 기하급수적으로 늘어난다**.

### 5.4 인수 객체

- 인수가 2~3개 이상 필요할 경우, 그 일부를 **의미 있는 객체로 묶을 수 있는지 고민**해야 한다.
- 인수를 묶는다고 해서 **눈속임이 되는 것은 아니며**, 오히려 **이름을 통해 개념을 명확히 표현**할 수 있게 된다.
- 이는 설계를 단순화하고 **함수의 의미를 더 잘 드러내는 좋은 방법**이다.

## 6. 부수 효과를 일으키지 마라

- 부수 효과(side effect)는 함수를 **하나의 목적**으로 작성한다고 해놓고, **몰래 다른 동작까지 수행하는 것**을 말한다.
- 대표적으로 **전역 변수나 인스턴스 상태를 변경하는 행위**, 또는 **입력 값에 영향을 주는 로직**이 부수 효과의 예다.
- 이런 부수 효과는 **시간적 결합(temporal coupling)** 이나 **순서 종속성(order dependency)** 을 초래하여 **디버깅을 어렵게 만들고 유지보수를 힘들게 만든다**.

### 6.1 예시 코드 (문제 있는 형태)

- checkPassword()는 비밀번호 검증만 수행해야 하지만, 세션 초기화라는 부수 효과를 동반하고 있다.
- 사용자는 checkPassword() 호출만으로는 세션에 영향을 줄 거라고 예상하기 어렵기 때문에 **기능 분리가 필요**하다.

```java
public class UserValidator {
    private Cryptographer cryptographer;
    private UserRepository userRepository;
    private Session session;

    public UserValidator(Cryptographer cryptographer, UserRepository userRepository, Session session) {
        this.cryptographer = cryptographer;
        this.userRepository = userRepository;
        this.session = session;
    }

    public boolean checkPassword(String username, String password) {
        User user = userRepository.findByName(username);
        if (user != null) {
            String encryptedPassword = cryptographer.encrypt(password);
            if (encryptedPassword.equals(user.getEncryptedPassword())) {
                return true;
            }
        }
        session.initialize(); // ❌ 부수 효과: 비밀번호 검증 실패 시 세션을 초기화함
        return false;
    }
}
```

### 6.2 개선 예시

- **함수의 책임을 분리**하고, **부수 효과를 호출하는 쪽에서 명시적으로 다루도록** 해야 한다.
- 깨끗한 함수는 예측 가능하고, 실행 전과 후의 상태 차이를 최소화해야 한다.

```java
// ✅ 개선된 코드 예시 - 책임을 분리하여 부수 효과 제거
public boolean checkPassword(String username, String password) {
    User user = userRepository.findByName(username);
    if (user == null) return false;

    String encryptedPassword = cryptographer.encrypt(password);
    return encryptedPassword.equals(user.getEncryptedPassword());
}
```

```
// ❗ 호출부에서 의도를 명확히 드러내도록 분리
if (!userValidator.checkPassword(username, password)) {
    session.initialize();
}
```

## 7. 명령과 조회를 분리하라

- 함수는 **무언가를 수행하거나, 값을 반환하거나 둘 중 하나만 해야 한다.**
- 즉, **객체 상태를 변경하는 명령(Command)** 이거나, **객체 정보를 반환하는 조회(Query)** 이어야 한다.
- 둘 다 수행하는 함수는 호출자에게 혼란을 준다. 호출자는 함수의 반환값을 이용할 수 없거나, 명령의 결과를 예측할 수 없게 된다.
- 예를 들어 `if (setUserName("Alice"))`처럼 `setUserName()`이 이름을 설정하고 동시에 boolean을 반환한다면, 이 함수가 **이름을 바꾸는 것인지**, **검증만 하는 것인지** 모호해진다.

## 8. 오류 코드보다 예외를 사용하라

- 명령 함수가 **오류 코드를 반환**하면, 이를 처리하기 위해 **호출자 쪽 코드에 조건문이 누적**된다.
- 이는 **명령과 조회의 분리 원칙**을 위반하며, **정상 흐름과 예외 흐름을 혼합**시켜 코드 가독성과 유지보수성을 해친다.

```java
if (deleteUser(user)) {
    // 삭제 성공 시
} else {
    // 삭제 실패 시
}
```

- 위와 같은 방식은 함수가 **실패할 수 있다는 사실을 호출자에게 떠넘기고**, 조건문을 통해 **오류를 일일이 점검**하게 만든다.
- 또 다른 문제는 어딘가에 **오류 코드 상수를 정의**해두고, 해당 값을 반환/비교하는 방식으로 **전역 의존성**이 생긴다는 점이다.
- 만약 **오류 코드 정의가 바뀐다면**, 이를 사용하는 코드 전체에 영향을 미쳐 **전체 재테스트가 필요**해질 수 있다.
- **반면, 예외를 사용하면 이러한 문제를 피할 수 있다.**
- 예외를 사용하면 **정상 흐름과 오류 흐름이 분리**되며, 호출자에게 **오류 처리 책임을 명확히 위임**할 수 있다.
- 또한 **try-catch 블록을 별도 함수로 분리**하면, 주 흐름과 오류 흐름을 **명확하게 나눌 수 있어 코드가 깔끔**해진다.
- **오류 처리도 하나의 작업**이다. 따라서 오류를 처리하는 함수는 **오류만 처리해야 한다.**

```java
public void deleteUserSafely(User user) {
    try {
        deleteUser(user);
    } catch (Exception e) {
        logError(e);
    }
}

private void logError(Exception e) {
    log.log(e.getMessage());
}
```

## 9. 함수를 작성하는 법

- 함수를 처음 작성할 때는 **길고 복잡**하다. 들여쓰기 단계도 많고, 중복된 루프도 많다. 인수 목록도 길고, **이름은 즉흥적이며**, **코드는 중복**된다.
- 하지만 그 **서투른 코드라도 빠짐없이 테스트하는 단위 테스트 케이스를 먼저 만든다.**
- 그런 다음 코드를 **다듬고**, **함수를 추출하고**, **이름을 바꾸고**, **중복을 제거**한다. 필요하면 **메서드를 줄이고 순서를 바꾸고**, **전체 클래스를 분리**하기도 한다.
- 이 모든 리팩터링 과정에서도 **코드는 항상 단위 테스트를 통과**해야 한다.
- 최종적으로는 위에서 설명한 **클린 코드의 규칙을 따르는 함수가 탄생**한다.

## 10. 마무리

- 모든 시스템은 특정 응용 분야를 표현하기 위해 **프로그래머가 설계한 도메인 특화 언어(Domain Specific Language, DSL)** 로 구성된다.
- 이 DSL에서 **함수는 동사**, **클래스는 명사**의 역할을 한다.
- 시스템을 **단순한 프로그램이 아닌 이야기로 바라보아야 한다.**
- 프로그래밍 언어는 **풍부하고 표현력 있는 이야기를 풀어내기 위한 수단**이며, 시스템의 모든 동작을 설명하는 함수 계층이 **그 이야기를 구성하는 언어의 일부**가 된다.
- **함수를 잘 만드는 것도 중요하지만**, 진짜 목적은 **‘시스템’이라는 이야기를 자연스럽고 명확하게 풀어가는 데 있다.**
