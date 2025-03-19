# TDD(Test Driven Development)

## 1. TDD(Test-Driven Development)

- **프로덕션 코드보다 테스트 코드를 먼저 작성하여, 테스트가 구현 과정을 주도하도록 하는 방법론이다**.
- **Red, Green, Refactoring** 단계로 진행된다.
  - **Red**: 실패하는 테스트 작성 → 구현부 없이 테스트를 진행한다.
  - **Green**: 테스트를 통과하는 최소한의 코딩 → 빠르게 테스트를 통과할 수 있도록 개발한다.
  - **Refactoring**: 구현 코드를 개선하며, 테스트 통과 상태를 유지한다.

## 2. TDD와 피드백

### 2.1 선 기능 구현, 후 테스트 작성 시 문제점

- 테스트 자체가 누락될 가능성이 있다.
- 특정 테스트 케이스(해피 케이스)만 검증할 가능성이 있다.
- 잘못된 구현을 다소 늦게 발견할 가능성이 있다.

### 2.2 선 테스트 작성, 후 기능 구현의 장점

- **복잡도가 낮은(유연하며 유지보수가 쉬운), 테스트 가능한 코드로 구현할 수 있다**.
- **쉽게 발견하기 어려운 엣지(Edge) 케이스를 놓치지 않도록 한다**.
- **구현에 대한 빠른 피드백을 받을 수 있다**.
- **과감한 리팩토링이 가능해진다**.

## 3. 관점의 변화

- 기존: **테스트는 구현부 검증을 위한 보조 수단이다**.
- TDD: **테스트와 상호 작용하며 발전하는 구현부이다**.
- **클라이언트 관점에서 피드백을 제공하는 Test-Driven 개발 방식이다**.

## 4. TDD 예제

- 단위 테스트에서 구현한 `CafeKiosk` 객체에 전체 음료의 금액을 더하는 `calculateTotalPrice()` 메서드를 TDD 방식으로 구현하려고 한다.

### 4.1 컴파일이 되도록 구현

- 아직 구현되지 않은 기능을 먼저 테스트 코드로 작성한다.
- 이 단계에서는 테스트가 실패(Red)해야 한다.

```java
// Test Code
@Test
void calculateTotalPrice() {

    CafeKiosk cafeKiosk = new CafeKiosk();
    Americano americano1 = new Americano();
    Americano americano2 = new Americano();

    cafeKiosk.add(americano1);
    cafeKiosk.add(americano2);

    int totalPrice = cafeKiosk.calculateTotalPrice();

    assertThat(totalPrice).isEqualTo(8000);
}
```

```java
// CafeKiosk class
public int calculateTotalPrice() {
    return 0;
}
```

### 4.2 하드코딩으로 실행은 가능하도록 변경

- 테스트를 통과하도록 최소한의 구현(Green)만 진행한다.
- 아직 실제 로직은 구현되지 않았으며, 테스트가 성공하도록 하드코딩된 값(8000)을 반환한다.

```java
// CafeKiosk class
public int calculateTotalPrice() {
    return 8000;
}
```

### 4.3 리팩토링 - 반복문 적용

- 실제 로직을 구현하며, 하드코딩된 값을 제거한다.
- 각 음료의 가격을 합산하는 반복문을 추가한다.

```java
// CafeKiosk class
public int calculateTotalPrice() {
    int totalPrice = 0;

    for(Beverage beverage : beverages) {
        totalPrice += beverage.getPrice();
    }

    return totalPrice;
}
```

### 4.4 리팩토링 - 스트림 적용

- 자바 스트림 API를 활용하여 더 간결하게 구현한다.
- 반복문 대신 `stream().mapToInt()`를 사용하여 가격을 합산한다.

```java
public int calculateTotalPrice() {
    return beverages.stream().mapToInt(Beverage::getPrice).sum();
}
```
