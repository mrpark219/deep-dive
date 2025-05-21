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
