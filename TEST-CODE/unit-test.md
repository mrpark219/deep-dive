# 단위 테스트

## 1. 단위 테스트

- **작은 코드 단위(클래스 or 메서드)를 독립적으로 검증하는 테스트**이다.
- 검증 속도가 빠르고, 안정적이다.

## 2. JUnit5

- **자바 단위 테스트를 위한 대표적인 테스트 프레임워크**이다.
- **XUnit 계열 프레임워크**로, Kent Beck이 개발하였다.
- **어노테이션 기반 테스트 실행(@Test, @BeforeEach, @AfterEach 등)을 지원**한다.

## 3. AssertJ

- **테스트 코드 작성을 보다 직관적으로 돕는 테스트 라이브러리**이다.
- **풍부한 API 제공**과 **메서드 체이닝 지원**을 통해 가독성을 높일 수 있다.
- `assertThat(actual).isEqualTo(expected);` 와 같은 표현식으로 **가독성이 뛰어난 테스트 코드 작성 가능**하다.

## 4. 참고 - 질문하기

- **테스트를 설계할 때, 암묵적이거나 아직 드러나지 않은 요구사항을 파악하는 것이 중요하다**.
- **테스트를 통해 시스템이 원하는 대로 동작하는지 명확히 검증할 수 있어야 한다**.

## 5. 테스트 케이스 세분화하기

- **해피 케이스(Happy Case)**: 정상적인 입력이 주어졌을 때, 예상대로 동작하는지를 검증한다.
- **예외 케이스(Exception Case)**: 예상치 못한 입력이나 상황이 발생했을 때, 적절한 예외 처리가 이루어지는지 검증한다.
- **경계값 테스트(Boundary Value Test)**: 범위(이상, 이하, 초과, 미만), 구간, 날짜 등 경계값에서의 동작을 검증한다.

## 6. 테스트하기 어려운 영역을 구분하고 분리하기

- **테스트가 가능한 구조로 코드를 설계하는 것이 중요하다**.
- **테스트하기 어려운 코드는 외부로 분리하면 테스트 가능성이 높아진다**.
- **테스트가 어려운 부분을 명확히 식별하고, 의존성을 줄이거나 대체할 방법을 고민해야 한다**.

## 7. 테스트하기 어려운 영역

- **관측할 때마다 결과가 달라지는 코드**
  - 현재 날짜/시간을 사용하는 코드
  - 랜덤 값 생성 코드
  - 전역 변수/함수에 의존하는 코드
  - 사용자 입력을 직접 받는 코드
- **외부 세계에 영향을 주는 코드**
  - 표준 출력(System.out.println)
  - 이메일/문자 메시지 발송
  - 데이터베이스에 직접 기록하는 코드
  - 네트워크 요청을 보내는 코드

## 8. 테스트하기 쉬운 코드 = 순수 함수(Pure Function)

- **같은 입력에 대해 항상 같은 결과를 반환하는 함수**이다.
- **외부 상태에 의존하지 않고, 부작용(side-effect)이 없는 코드**이다.
- **테스트가 쉽고, 예측 가능하며, 유지보수성이 뛰어나다**.
- 대표적인 예: `Math.max(a, b)` → 같은 입력이 주어지면 항상 같은 결과가 반환된다.

## 9. 단위 테스트 예제

### 9.1 기본 단위 테스트 예제

- `Americano` 객체의 `getName()`, `getPrice()` 메서드가 올바르게 동작하는지 확인하는 단위 테스트 코드이다.
- **JUnit5와 AssertJ를 사용하여 검증**한다.
- `getName()`함수를 호출 했을 때 `assertEquals()`(JUnit5)와 `assertThat()`(AssertJ)을 사용하여 "아메리카노"가 반환되는지 검증한다.
- `getPrice()` 함수를 호출 했을 때 4000이 반환되는지 검증한다.

```java
public interface Beverage {

    String getName();

    int getPrice();
}
```

```java
public class Americano implements Beverage {

    @Override
    public String getName() {
        return "아메리카노";
    }

    @Override
    public int getPrice() {
        return 4000;
    }
}
```

```java
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertEquals;

class AmericanoTest {

    @Test
    void getName() {

        Americano americano = new Americano();

        assertEquals("아메리카노", americano.getName()); // jUnit
        assertThat(americano.getName()).isEqualTo("아메리카노"); // assertJ
    }

    @Test
    void getPrice() {

        Americano americano = new Americano();

        assertThat(americano.getPrice()).isEqualTo(4000);
    }
}
```

### 9.2 예외 발생 테스트(경계값 테스트) 예제

- `CafeKiosk`는 음료를 관리하는 객체이다.
- 여러 개의 `Beverage` 객체(음료)를 저장할 수 있으며, **음료를 추가하는 기능**을 제공한다.
- 내부적으로 `beverages`라는 리스트(List)를 가지고 있으며, `add()` 메서드를 통해 리스트에 음료를 추가한다.
- `CafeKiosk` 객체의 `add()` 메서드에서 `count` 인자가 **0 이하**일 경우 예외를 발생시키는지 테스트한다.

```java
public class CafeKiosk {

    private final List<Beverage> beverages = new ArrayList<>();

    public void add(Beverage beverage, int count) {

        if(count <= 0) {
            throw new IllegalArgumentException("음료는 1잔 이상 주문하실 수 있습니다.");
        }

        for(int i = 0; i < count; i++) {
            beverages.add(beverage);
        }
    }

    public List<Beverage> getBeverages() {
        return beverages;
    }
}
```

```java
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;

class CafeKioskTest {

    @Test
    void addSeveralBeverages() {

        CafeKiosk cafeKiosk = new CafeKiosk();
        Americano americano = new Americano();

        cafeKiosk.add(americano, 2); // 0 이상을 입력해서 정상 동작 확인

        assertThat(cafeKiosk.getBeverages().get(0)).isEqualTo(americano);
        assertThat(cafeKiosk.getBeverages().get(1)).isEqualTo(americano);
    }

    @Test
    void addZeroBeverages() {

        CafeKiosk cafeKiosk = new CafeKiosk();
        Americano americano = new Americano();

        assertThatThrownBy(() -> cafeKiosk.add(americano, 0)) // 0을 입력해서 예외 처리 확인
            .isInstanceOf(IllegalArgumentException.class) // 예외 타입 검증
            .hasMessage("음료는 1잔 이상 주문하실 수 있습니다."); // 예외 메시지 검증
    }
}
```

### 9.3 테스트하기 어려운 영역을 분리하는 테스트 예제

- 첫 번째 `createOrder()` 메서드는 **현재 시간을 직접 호출**하여 주문 시간을 검증한다.
  - `LocalDateTime.now()`를 호출하는 시점마다 **다른 값이 반환되므로 테스트하기 어렵다**.
  - 테스트를 실행할 때마다 실행 시점이 다르므로 **예측 가능한 결과를 보장하기 어렵다**.
  - 테스트를 할 때 **현재 시간을 원하는 값으로 고정할 수 없다**.
- 두 번째 `createOrder()` 메서드는 **현재 시간을 메서드 안에서 직접 불러오지 않고, 매개변수로 전달**한다.
  - `currentDateTime`을 파라미터로 받아서 테스트할 때 **원하는 시간 값을 전달할 수 있다**.
  - 테스트 환경에서는 현재 시간을 **고정된 값**으로 설정할 수 있으므로 **예측 가능한 테스트 결과를 보장**할 수 있다.
  - 외부에서 값을 주입받기 때문에 **시간과 관련된 로직을 독립적으로 테스트할 수 있다**. 메서드 밖에서 입력을 받아서 기능을 실행하고 있다.

```java
// CafeKiosk.class - 1
private static final LocalTime SHOP_OPEN_TIME = LocalTime.of(10, 0);
private static final LocalTime SHOP_CLOSE_TIME = LocalTime.of(22, 0);

public Order creatOrder() {

    LocalDateTime currentDateTime = LocalDateTime.now();
    LocalTime currentTime = currentDateTime.toLocalTime();

    if(currentTime.isBefore(SHOP_OPEN_TIME) || currentTime.isAfter(SHOP_CLOSE_TIME)) {
        throw new IllegalArgumentException("주문 시간이 아닙니다. 관리자에게 문의하세요.");
    }

    return new Order(currentDateTime, beverages);
}
```

```java
// CafeKiosk.class - 2
private static final LocalTime SHOP_OPEN_TIME = LocalTime.of(10, 0);
private static final LocalTime SHOP_CLOSE_TIME = LocalTime.of(22, 0);

public Order creatOrder(LocalDateTime currentDateTime) {

    LocalTime currentTime = currentDateTime.toLocalTime();

    if(currentTime.isBefore(SHOP_OPEN_TIME) || currentTime.isAfter(SHOP_CLOSE_TIME)) {
        throw new IllegalArgumentException("주문 시간이 아닙니다. 관리자에게 문의하세요.");
    }

    return new Order(currentDateTime, beverages);
}
```
