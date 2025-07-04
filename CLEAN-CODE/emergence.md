# 창발성

## 1. 창발적 설계로 깔끔한 코드를 구현하자

- 대부분의 개발자는 켄트 벡이 제시한 **단순한 설계 규칙 네 가지**가 소프트웨어 설계 품질을 **크게 향상시킨다고 믿는다**.
- 아래 규칙은 **중요도 순**으로 나열한다.

> 1. **모든 테스트를 실행한다**
> 2. **중복을 없앤다**
> 3. **프로그래머 의도를 표현한다**
> 4. **클래스와 메서드 수를 최소로 줄인다**

## 2. 단순한 설계 규칙 1: 모든 테스트를 실행하라

- 설계는 시스템이 **의도한 대로 동작하도록 보장해야 한다**.
- 문서로 아무리 정밀하게 설계해도 시스템이 제대로 동작하는지 **검증할 방법이 없다면 의미가 없다**.
- 모든 테스트 케이스를 항상 통과하는 시스템은 **테스트 가능한 시스템이다**.
- 테스트 가능한 시스템을 만들기 위해 노력하다 보면 **자연스럽게 설계 품질이 향상된다**.
- 결합도가 높으면 테스트가 어렵기 때문에 **테스트를 많이 작성할수록 DIP, DI, 인터페이스, 추상화 등 설계 원칙을 적용하게 된다**.

## 3. 단순한 설계 규칙 2~4: 리팩터링

- 테스트 케이스를 모두 작성했다면 코드를 **점진적으로 리팩터링할 수 있다**.
- 테스트가 있으므로 리팩터링 과정에서 시스템이 망가질 걱정은 없다.
- 이 단계에서는 중복을 제거하고, 의도를 표현하고, 클래스와 메서드 수를 줄인다.

## 4. 중복을 없애라

- **중복은 설계의 적이다**.
- 중복은 **작업량 증가, 오류 가능성, 불필요한 복잡도**를 야기한다.
- 똑같은 코드뿐 아니라 **비슷한 코드도 리팩터링의 대상이다**.
- **구현 중복**도 제거해야 한다.
- **소규모 재사용을 익혀야 대규모 재사용도 가능하다**.

## 5. 표현하라

- 자신만 이해할 수 있는 코드를 짜는 것은 쉬우나, **다른 사람도 쉽게 이해할 수 있어야 한다**.
- 소프트웨어 프로젝트 비용 대부분은 **유지보수에 사용된다**.
- 코드는 **프로그래머의 의도를 명확하게 표현해야 한다**.
- 의도를 잘 표현하면 **결함이 줄고 유지보수가 쉬워진다**.
- 다음을 실천해야 한다:
  - **이름과 기능이 일치하도록 적절한 이름을 선택한다**.
  - **작은 클래스와 함수로 구성해 이해를 돕는다**.
  - **디자인 패턴 등 표준 명칭을 사용해 의사소통을 원활하게 한다**.
  - **테스트 케이스를 문서처럼 작성해 사용 예시로 활용한다**.
- **가장 중요한 것은 읽기 쉬운 코드를 작성하려는 지속적인 노력이다**.

## 6. 클래스와 메서드 수를 최소로 줄여라

- SRP, 중복 제거, 의도 표현 등 **기본 원칙을 지나치게 추구하면 오히려 복잡도를 유발한다**.
- 클래스와 메서드 수를 무작정 늘리는 방식은 피해야 한다.
- 예를 들어 **모든 클래스에 인터페이스를 강제하는 정책은 비효율적이다**.
- 목표는 **작은 함수와 클래스 크기뿐 아니라 전체 시스템 크기도 작게 유지하는 것이다**.
- 하지만 이 규칙은 **우선순위상 가장 낮으며**, 테스트 작성, 중복 제거, 의도 표현이 **더 중요하다**.

## 7. 마무리

- 경험을 완전히 대체할 수 있는 기법은 없다.
- 그러나 **단순한 설계 규칙을 따르면 우수한 설계 원칙을 자연스럽게 적용할 수 있다**.
