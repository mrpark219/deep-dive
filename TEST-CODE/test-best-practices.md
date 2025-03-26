# 효율적인 테스트 작성 방법

## 1. 한 문단에 한 주제

- **테스트 코드도 글쓰기와 같다고 본다면, 하나의 테스트는 하나의 문단이 되어야 한다**.
- 한 테스트는 **오직 하나의 목적만 검증해야 한다**.
- 따라서 `@DisplayName`을 **한 문장으로 명확하게 표현할 수 있는 형태**가 되어야 한다.
- **하나의 테스트에 여러 논리 구조나 조건이 들어가면 가독성과 유지보수가 어려워진다**.
- 아래 예시는 상품 타입이 재고 관련 타입인지 체크하는 테스트이다. **모든 타입에 대한 테스트를 하나의 테스트에 넣어버려서 테스트 코드가 복잡해지고 논리 구조가 포함되는 문제를 가지고 있다**.

```java
// 안 좋은 예시
@DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
@Test
void containsStockType() {

    // given
    ProductType[] productTypes = ProductType.values();

    for(ProductType productType : productTypes) {

        if(productType == ProductType.HANDMADE) {
            // when
            boolean result = ProductType.containsStockType(productType);

            // then
            assertThat(result).isFalse();
        }

        if(productType == ProductType.BAKERY || productType == ProductType.BOTTLE) {
            // when
            boolean result = ProductType.containsStockType(productType);

            // then
            assertThat(result).isTrue();
        }
    }
}
```

## 2. 완벽하게 제어하기

- **테스트 환경은 예측 가능하고 완벽하게 제어할 수 있어야 한다**.
- **랜덤 값, 현재 시간처럼 제어가 어려운 요소들은 외부에서 주입하거나 상위 계층으로 분리하는 것이 좋다**.
- **외부 시스템과 연동되는 부분은 반드시 Mock으로 대체하여 테스트 환경에서 독립성을 확보해야 한다**.
- 아래 예시의 첫 번째 코드처럼 `createOrder()` 메서드 내부에서 `LocalDateTime.now()`를 직접 호출하면, 테스트 실행 시점에 따라 동작이 달라질 수 있으므로 테스트의 일관성과 신뢰성이 떨어진다. 이 경우에는 테스트 시점마다 다른 현재 시간이 사용되기 때문에 **재현 가능한 테스트를 작성하기 어렵다**.
- 반면 예시의 두 번째 코드처럼 현재 시간을 외부에서 주입받도록 변경하면 테스트 코드에서 명확한 시점을 지정해 테스트할 수 있으므로, 제어 가능한 테스트가 가능해진다.
- 테스트는 **항상 같은 입력에 대해 같은 출력을 기대**할 수 있어야 하며, 이를 위해 제어 가능한 설계가 필수적이다.

```java
public Order creatOrder() {

    LocalDateTime currentDateTime = LocalDateTime.now();
    LocalTime currentTime = currentDateTime.toLocalTime();

    if(currentTime.isBefore(SHOP_OPEN_TIME) || currentTime.isAfter(SHOP_CLOSE_TIME)) {
        throw new IllegalArgumentException("주문 시간이 아닙니다. 관리자에게 문의하세요.");
    }

    return new Order(currentDateTime, beverages);
}

public Order creatOrder(LocalDateTime currentDateTime) {

    LocalTime currentTime = currentDateTime.toLocalTime();

    if(currentTime.isBefore(SHOP_OPEN_TIME) || currentTime.isAfter(SHOP_CLOSE_TIME)) {
        throw new IllegalArgumentException("주문 시간이 아닙니다. 관리자에게 문의하세요.");
    }

    return new Order(currentDateTime, beverages);
}
```

## 3. 테스트 환경의 독립성을 보장하자

- 테스트는 다른 테스트나 외부 요소에 의존하지 않고, 독립적으로 실행 가능해야 한다.
- **Given 절에서는 테스트 환경을 조성하는 데 집중하고, When/Then 절에 영향을 주는 요소는 없어야 한다**.
- 테스트 준비 과정에서 다른 API나 함수를 사용하면 테스트 간 결합도가 높아지고, 테스트 신뢰성이 떨어진다.
- **객체 생성 시 생성자나 빌더를 활용하는 것이 좋으며, 목적이 내포된 팩토리 메서드는 피하는 것이 바람직하다**.
- 아래 예제는 테스트 환경이 불완전하게 구성되어 있는 경우이다. `stock1.deductQuantity(3)` 코드가 **테스트를 준비하는 Given 절에서 실행되지만**, 이 메서드가 실패할 수 있는 로직(재고 부족)을 포함하고 있기 때문에 **테스트 목적과는 다른 이유로 실패할 수 있다**.
- **좋은 테스트는 실패했을 때 그 이유를 명확하게 전달해야 하며, 테스트 목적 외의 코드가 개입되지 않도록 독립성을 보장해야 한다**.

```java
@DisplayName("재고가 부족한 상품으로 주문을 생성하려는 경우 예외가 발생한다.")
@Test
void createOrderWithNoStock() {

    // given
    LocalDateTime registeredDateTime = LocalDateTime.now();

    Product product1 = createProduct(BOTTLE, "001", 1000);
    Product product2 = createProduct(BAKERY, "002", 3000);
    Product product3 = createProduct(HANDMADE, "003", 5000);

    productRepository.saveAll(List.of(product1, product2, product3));

    Stock stock1 = Stock.create("001", 2);
    Stock stock2 = Stock.create("002", 2);
    // 재고 수량을 감소시키는 함수가 실행되면서 테스트가 실패한다.
    stock1.deductQuantity(3);

    stockRepository.saveAll(List.of(stock1, stock2));

    OrderCreateServiceRequest request = OrderCreateServiceRequest.builder()
            .productNumbers(List.of("001", "001", "002", "003"))
            .build();

    // when
    // then
    assertThatThrownBy(() -> orderService.createOrder(request, registeredDateTime))
            .isInstanceOf(IllegalArgumentException.class)
                    .hasMessage("재고가 부족한 상품이 있습니다.");
}
```

## 4. 테스트 간 독립성을 보장하자

- 테스트는 서로의 상태에 영향을 주지 않고 독립적으로 실행되어야 한다.
- **하나의 테스트가 공유 자원의 상태를 변경하면, 다른 테스트 결과가 달라질 수 있다**.
- **테스트 실행 순서에 따라 성공하거나 실패하는 테스트는 신뢰할 수 없다**.
- **테스트는 언제, 어떤 순서로 실행되더라도 항상 같은 결과를 보장해야 한다**.
- 아래 예제에서 `Stock` 객체를 static 필드로 선언하고 두 테스트가 이를 공유하고 있다. 두 테스트가 공유 자원의 상태 값을 통해서 테스트를 진행하고 있기 때문에 실행 순서에 따라 테스트가 성공하거나 실패하게 되는 위험한 구조이다.

```java
private static final Stock stock = Stock.create("001", 1);

@DisplayName("재고의 수량이 제공된 수량보다 작은지 확인한다.")
@Test
void isQuantityLessThan() {

    // given
    int quantity = 2;

    // when
    boolean result = stock.isQuantityLessThan(quantity);

    // then
    assertThat(result).isTrue();
}

@DisplayName("재고를 주어진 개수만큼 차감할 수 있다.")
@Test
void deductQuantity() {

    // given
    int quantity = 1;

    // when
    stock.deductQuantity(quantity);

    // then
    assertThat(stock.getQuantity()).isZero();
}
```

## 5. Text Fixture

- **Fixture는 테스트를 위해 고정된 상태를 만들어주는 객체 또는 설정을 의미한다**.
- `given` 절에서 테스트 데이터를 생성하는 부분이 이에 해당한다.
- JUnit에서는 아래와 같은 어노테이션을 통해 테스트 실행 전후의 동작을 제어할 수 있다.

  - `@BeforeAll`: 클래스 전체에서 테스트 실행 전에 한 번만 실행된다.
  - `@BeforeEach`: 각 테스트 메서드 실행 전에 매번 실행된다.
  - `@AfterAll`: 클래스 전체 테스트 실행 후 한 번만 실행된다.
  - `@AfterEach`: 각 테스트 메서드 실행 후에 매번 실행된다.

### 5.1 Fixture 활용 시 주의할 점

- `@BeforeEach`와 같이 픽스처를 통해 공통 데이터를 미리 설정하는 경우 **공유 자원의 위험**이 발생할 수 있다.
- **모든 테스트에서 해당 픽스처를 반드시 사용하는 것이 아니라면, 각 테스트 내부에 `given` 절을 직접 작성하는 것이 문서로서의 가독성이 높다**.
- **테스트가 길어질수록 셋업 코드를 확인하기 위해 위아래로 스크롤을 반복해야 하므로 가독성이 떨어진다**.
- 테스트에 필요한 객체만을 생성하고, 불필요한 엔티티는 포함시키지 않아야 한다.
- **각 테스트는 독립적이고, 필요한 조건만을 갖춘 상태에서 실행되어야 한다**.

### 5.2 @BeforeEach 사용 기준

- 테스트 내용을 이해하는 데 픽스처 코드가 없어도 괜찮은가?
- 픽스처를 수정했을 때 다른 테스트에 영향을 주지 않는가?
- 이 두 질문에 **모두 'Yes'라면** `@BeforeEach`를 사용해도 괜찮다.

### 5.3 Fixture 생성 팁

- **데이터 생성이 반복될 경우, 별도의 메서드로 추출해서 사용한다**.
- 해당 테스트 클래스에서만 사용하는 값들로 메서드 파라미터를 구성한다.
- 프로젝트 전반에서 공통적으로 사용하는 빌더들을 따로 모으고 싶지만, 테스트마다 파라미터가 달라져서 빌더만 불필요하게 늘어나는 문제가 발생할 수 있다. 따라서 **테스트 클래스 내에서 해결할 수 있는 건 내부에서 처리하는 것이 더 효율적이다**.

### 5.4 잘못된 방식

- `data.sql` 등을 활용해 테스트용 데이터를 미리 주입하는 방식은 **데이터 파편화**와 **관리 포인트 증가**라는 문제를 초래하므로 지양해야 한다.

## 6. Text Fixture 데이터 클렌징 (deleteAll() vs deleteAllInBatch())

### 6.1 deleteAll()

- 동작 방식
  - `select` 쿼리를 먼저 실행하고, `where` 조건에 ID를 넣어서 개별적으로 `delete`를 수행한다.
  - 연관 관계가 있는 테이블의 데이터를 먼저 조회한 후 삭제한다.
- 장점
  - 연관 관계가 걸린 테이블에 대해 **JPA가 안전하게 삭제 순서를 보장해준다**.
- 단점
  - **쿼리 수가 많고, 성능이 떨어질 수 있다**.
  - 대용량 테스트에서는 느려질 수 있다.
  - 연관 관계가 단방향으로 매핑되어 있는 경우에는 JPA에서 인식할 수 없어 삭제 순서를 보장해주지 않는다.

### 6.2 deleteAllInBatch()

- 동작 방식
  - `select` 없이 해당 테이블 전체에 대해 **한 번의 `delete` 쿼리**를 실행한다.
  - 벌크 연산(Bulk Operation)을 수행한다.
- 장점
  - **성능이 빠르다**. (쿼리 1번으로 전체 삭제)
  - 대용량 데이터를 삭제할 때 유리하다.
- 단점
  - **삭제 순서를 JPA가 보장하지 않는다**.
  - 외래키 제약 조건이 있는 경우, 삭제 순서를 잘못 지정하면 **무결성 제약 위반 오류가 발생할 수 있다**.

### 6.3 사용 권장 기준

- 일반 단위 테스트에서는 `@Transactional`을 활용하여 테스트 후 롤백하는 방식이 가장 안전하다. 하지만 `@Transactional`의 사이드 이펙트를 알고 사용해야 한다.
- Spring Batch 테스트나 트랜잭션의 경계가 복잡한 테스트에서는 `deleteAllInBatch()`를 사용하고, 삭제 순서를 명확히 지정해야 한다.
- 테스트 간 **독립성과 명확한 상태 초기화**를 위해 **테스트 후 `@AfterEach`에서 클렌징을 명시적으로 수행하는 것이 좋다.**
- 아래 예시는 테스트가 끝난 후 데이터 정리를 수행하는 코드이다.

```java
@AfterEach
void tearDown() {
    orderProductRepository.deleteAllInBatch();
    orderRepository.deleteAllInBatch();
    productRepository.deleteAllInBatch();
    mailSendHistoryRepository.deleteAllInBatch();
}
```

## 7. @ParameterizedTest

- 테스트에서는 **하나의 테스트 케이스에 여러 개의 검증 로직이나 분기 조건을 넣으면 안 된다**.
- 테스트 케이스가 많아지면 **입력 값만 다른 반복 테스트를 하게 되는 경우가 많은데**, 이때 유용하게 사용할 수 있는 것이 `@ParameterizedTest`이다.
- **동일한 테스트 로직에 대해 여러 입력 값으로 반복 테스트를 수행할 수 있도록 도와주는 JUnit5 기능이다**.

### 7.1 @CsvSource

- **입력 값들을 CSV 형식으로 나열하여 파라미터로 주입**할 수 있다.
- 각 값은 쉼표(`,`)로 구분하며, 각 테스트에 사용할 **여러 조합의 인자들을 간단하게 정의**할 수 있다.

```java
@DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
@CsvSource({"HANDMADE,false", "BOTTLE,true", "BAKERY,true"})
@ParameterizedTest
void containsStockTypeParameterized1(ProductType productType, boolean expected) {

    //when
    boolean result = ProductType.containsStockType(productType);

    //then
    assertThat(result).isEqualTo(expected);
}
```

### 7.3 @MethodSource

- 복잡한 로직이나 여러 줄의 인자를 구성해야 하는 경우, 메서드에서 인자들을 반환받아 주입하는 방식이다.
- 동적 생성이나 조건부 조합이 필요할 때 유용하게 사용할 수 있다.

```java
private static Stream<Arguments> provideProductTypesForCheckingStockType() {
    return Stream.of(
        Arguments.of(ProductType.HANDMADE, false),
        Arguments.of(ProductType.BOTTLE, true),
        Arguments.of(ProductType.BAKERY, true)
    );
}

@DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
@MethodSource("provideProductTypesForCheckingStockType")
@ParameterizedTest
void containsStockTypeParameterized2(ProductType productType, boolean expected) {

    // when
    boolean result = ProductType.containsStockType(productType);

    // then
    assertThat(result).isEqualTo(expected);
}
```

## 8. @DynamicTest

- 일반적인 테스트는 정적인 테스트로, 컴파일 시점에 고정된 테스트 케이스를 기반으로 실행된다.
- 하지만 **여러 시나리오를 하나의 테스트 메서드에서 유연하게 구성하거나**, **동적으로 테스트 케이스를 생성하고 싶을 때**는 `@DynamicTest`를 사용할 수 있다.
- JUnit5에서 제공하는 기능으로, 테스트 메서드가 하나의 테스트가 아니라 **여러 개의 테스트를 생성하는 팩토리 역할을 수행하게 만든다**.

### 8.1 사용 방법

- `@TestFactory`를 사용해 테스트 메서드를 작성한다.
- 테스트 메서드는 `Collection<DynamicTest>` 또는 `Stream<DynamicTest>` 등의 타입을 반환한다.
- 각각의 `DynamicTest`는 테스트 이름과 실행 로직을 람다로 정의한다.
- **동적으로 테스트 케이스를 생성할 수 있어 다양한 상황을 한 메서드에서 처리할 수 있다**.

### 8.2 언제 사용하는가?

- **공유 객체를 기반으로 연속적인 테스트 시나리오를 실행해야 할 때 유용하다**.
- 예: 어떤 **초기 상태를 기반으로 다양한 사용자 행위를 순차적으로 시뮬레이션**해야 할 때 사용.
- 단, 공유 객체 사용으로 인해 테스트 간의 순서에 따라 영향을 주는 부작용이 발생할 수 있으므로 사용 시 주의가 필요하다.
- 아래 예시에서는 하나의 `Stock` 객체를 기반으로 2개의 테스트 시나리오를 구성하고 있다. 첫 번째 테스트는 수량을 정상적으로 차감하는 경우이고, 두 번째 테스트는 수량이 0인 상태에서 추가 차감을 시도하여 예외가 발생하는 경우이다. 동일한 `Stock` 객체에서 2번의 테스트가 진행되기 떄문에 순서에 의존하게 된다.

```java
@DisplayName("재고 차감 시나리오")
@TestFactory
Collection<DynamicTest> stockDeductionDynamicTest() {

    // given
    // stock을 공유해서 사용한다.
    Stock stock = Stock.create("001", 1);

    return List.of(
        DynamicTest.dynamicTest("재고를 주어진 개수만큼 차감할 수 있다.", () -> {

            // given
            int quantity = 1;

            // when
            stock.deductQuantity(quantity);

            // then
            assertThat(stock.getQuantity()).isZero();
        }),
        DynamicTest.dynamicTest("재고보다 많은 수의 수량으로 차감 시도하는 경우 예외가 발생한다.", () -> {

            // given
            int quantity = 1;

            // when // then
            assertThatThrownBy(() -> stock.deductQuantity(quantity))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessage("차감할 재고 수량이 없습니다.");
        })
    );
}
```
