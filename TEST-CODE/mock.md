# Mock

## 1. Test Double(테스트 대역)

### 1.1 Dummy

- 특징: 아무 것도 하지 않는 객체
- 용도: 테스트에서 단지 객체를 넘기기 위해 사용된다. 실제로 아무런 동작을 하지 않으며, 데이터 전달 용도로만 존재한다.

### 1.2 Fake

- 특징: 단순한 형태로 동일한 기능을 수행하나, 프로덕션에서 쓰기에는 부족한 객체
- 용도: 실제 객체처럼 동작하지만, 성능이나 안정성 면에서 부족한 기능을 가진 객체이다. 예를 들어, 테스트 환경에서만 사용하는 `FakeRepository` 같은 객체가 있다.

### 1.3 Stub(상태 검증, State Verification)

- 특징: 테스트에서 요청한 것에 대해 미리 준비한 결과를 제공하는 객체. 그 외에는 응답하지 않는다.
- 용도: 특정 입력에 대해 미리 정의된 출력을 반환하며, 주로 **상태 검증**에 사용된다. 외부 시스템과의 상호작용을 단순화하기 위해 사용된다.
- 상태 검증(State Verification): 테스트에서 요청한 것에 대해 미리 준비한 결과를 제공하는 객체이다. 그 외에는 응답하지 않는다.

### 1.4 Spy

- 특징: Stub이면서 호출된 내용을 기록하여 보여줄 수 있는 객체. 일부는 실제 객체처럼 동작시키고 일부만 Stubbing 할 수 있다.
- 용도: 일부 메서드는 실제로 호출하게 하고, 다른 메서드는 스텁으로 처리하여 호출 내역을 기록할 수 있다.

### 1.5 Mock(행위 검증, Behavior Verification)

- 특징: 행위에 대한 기대를 명세하고, 그에 따라 동작하도록 만들어진 객체
- 용도: 테스트 대상 객체의 **행위 검증**에 사용된다. 특정 메서드 호출이 예상대로 이루어졌는지, 호출 횟수는 몇 번이었는지 등을 검증하는 데 사용된다.
- 행위 검증(Behavior Verification): 행위에 대한 기대를 명세하고, 그에 따라 동작하도록 만들어진 객체이다.

### 1.6 Stub 예시

```java
@SpringBootTest
class OrderStatisticsServiceTest {

    @Autowired
    private OrderStatisticsService orderStatisticsService;

    @MockBean
    private MailSendClient mailSendClient;

    @DisplayName("결제 완료 주문들을 조회하여 매출 통계 메일을 전송한다.")
    @Test
    void sendOrderStatisticsMail() {

        // given
        LocalDateTime now = LocalDateTime.of(2024, 2, 5, 0, 0);

        Product product1 = createProduct(HANDMADE, "001", 1000);
        Product product2 = createProduct(HANDMADE, "002", 2000);
        Product product3 = createProduct(HANDMADE, "003", 3000);
        List<Product> products = List.of(product1, product2, product3);

        productRepository.saveAll(products);

        Order order1 = createPaymentCompletedOrder(LocalDateTime.of(2024, 2, 4, 23, 59, 59), products);
        Order order2 = createPaymentCompletedOrder(now, products);
        Order order3 = createPaymentCompletedOrder(LocalDateTime.of(2024, 2, 5, 23, 59, 59), products);
        Order order4 = createPaymentCompletedOrder(LocalDateTime.of(2024, 2, 6, 0, 0), products);

        // stubbing
        when(mailSendClient.sendEmail(any(String.class), any(String.class), any(String.class), any(String.class)))
            .thenReturn(true);

        // when
        boolean result = orderStatisticsService.sendOrderStatisticsMail(LocalDate.of(2024, 2, 5), "test@test.com");

        // then
        assertThat(result).isTrue();

        List<MailSendHistory> histories = mailSendHistoryRepository.findAll();
        assertThat(histories).hasSize(1)
            .extracting("content")
            .contains("총 매출 합계는 12000원 입니다.");
    }
}
```
