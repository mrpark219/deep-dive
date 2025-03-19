# 테스트 문서화하기

## 1. 테스트는 문서다

- 테스트 코드는 **프로덕션 기능을 설명하는 문서 역할**을 한다.
- 다양한 테스트 케이스를 통해 프로덕션 코드를 이해하는 시각과 관점을 보완할 수 있다.
- 한 사람이 과거에 고민했던 내용을 팀 차원에서 공유하고, 모두의 자산으로 활용할 수 있다.
- 우리는 항상 팀으로 협업하며, **테스트 코드는 지식 공유의 중요한 도구**가 된다.

## 2. `@DisplayName`을 섬세하게 작성하기

- 명사 나열보다 **문장 형태로 작성하는 것이 좋다**.
  - A이면 B이다.
  - A이면 B가 아니고 C다.
- `"~테스트"` 같은 표현을 지양하고, **테스트의 목적과 결과를 명확하게 기술**해야 한다.
- 메서드 자체의 관점보다 **도메인 정책 관점에서 서술**해야 한다.
- 테스트의 성공 또는 실패 같은 현상 중심의 기술은 피하고, **목적과 기대하는 동작을 표현**해야 한다.

## 3. BDD(Behavior Driven Development) 스타일로 작성하기

- TDD에서 파생된 개발 방법으로, 함수 단위의 테스트에 집중하기보다, 시나리오에 기반한 **테스트케이스 자체에 집중**하여 테스트하는 방식이다.
- 개발자가 아닌 사람도 이해할 수 있도록 **추상화 수준을 높이는 것이 좋다**.

### 3.1 Given/When/Then 패턴

- **Given**: 시나리오 진행에 필요한 모든 준비 과정(객체, 값, 상태 등) → 어떤 환경에서
- **When**: 시나리오 행동 진행 → 어떤 행동을 진행했을 때
- **Then**: 시나리오 진행에 대한 결과 명시, 검증 → 어떤 상태 변화가 일어난다

## 4. BDD 예시

- 기존에 작성한 `calculateTotalPrice()` 메서드를 **Given-When-Then 패턴**을 활용하여 개선한다.
- **`@DisplayName`을 활용하여 테스트 목적을 명확하게 기술**한다.

```java
@DisplayName("주문 목록에 담긴 상품들의 총 금액을 계산할 수 있다.")
@Test
void calculateTotalPrice() {

    // given - 주문 목록에 아메리카노 2개를 추가
    CafeKiosk cafeKiosk = new CafeKiosk();
    Americano americano1 = new Americano();
    Americano americano2 = new Americano();

    cafeKiosk.add(americano1);
    cafeKiosk.add(americano2);

    // when - 총 금액 계산
    int totalPrice = cafeKiosk.calculateTotalPrice();

    // then - 총 금액이 기대한 값과 일치하는지 검증
    assertThat(totalPrice).isEqualTo(8000);
}
```
