# 도메인 모델 시작하기

## 1. 도메인

- 소프트웨어로 해결하고자 하는 문제 영역을 **도메인**이라고 한다.
- 하나의 도메인은 다시 여러 개의 **하위 도메인**으로 나눌 수 있다.
- 특정 도메인을 위한 소프트웨어라고 해서, 관련된 모든 기능을 직접 구현해야 하는 것은 아니다.
- 도메인마다 고정된 하위 도메인이 존재하는 것은 아니다.
- 하위 도메인을 어떻게 구성할지는 **상황에 따라 달라질 수 있다**.

## 2. 도메인 전문가와 개발자 간 지식 공유

- 온라인 홍보나 정산 같은 각 도메인 영역에는 해당 분야의 **전문가**가 있다. 이들은 자신의 지식과 경험을 바탕으로 필요한 기능 개발을 요구한다.
- 개발자는 이러한 요구사항을 분석하고 설계한 뒤, 코드를 작성하고 테스트하여 배포하는 역할을 한다.
- 이 과정에서 코드를 작성하기에 앞서, **요구사항을 올바르게 이해하는 것이 매우 중요하다**.
- 요구사항을 제대로 이해하는 가장 좋은 방법은 개발자와 전문가가 **직접 소통**하는 것이다.
- 따라서 개발자 역시 **도메인 전문가 수준은 아니더라도, 관련된 지식을 갖추고 있어야 한다**.

## 3. 도메인 모델

- **도메인 모델**은 특정 도메인을 개념적으로 표현한 것이다.
- 도메인 모델을 사용하면 여러 관계자들이 **모두 동일한 관점에서 도메인을 이해하고 지식을 공유**할 수 있다.
- 보통 도메인은 기능과 데이터로 구성되는데, **객체 모델**은 이 두 가지를 함께 표현할 수 있어 도메인을 모델링하기에 적합하다.
- 반드시 클래스 다이어그램 같은 **UML 표기법으로만 모델을 표현해야 하는 것은 아니다**. 예를 들어, 관계가 중요한 도메인이라면 그래프를 사용할 수도 있다.
- 도메인 모델은 도메인을 이해하기 위한 **개념 모델**이므로, 실제 코드를 작성하기 위해서는 별도의 **구현 모델**이 필요하다.
- 여러 하위 도메인을 하나의 모델로 표현하면 안 된다. **각 하위 도메인마다 별도의 모델을 만들어야** 용어의 의미가 명확해지기 때문이다.

## 4. 도메인과 모델 패턴

- 일반적인 소프트웨어 아키텍처는 **표현, 응용, 도메인, 인프라스트럭처**의 네 가지 계층으로 구성된다.
- **표현(Presentation)** 계층은 사용자의 요청을 처리하고 정보를 보여주는 사용자 인터페이스(UI)를 담당한다. 여기서 사용자는 사람뿐만 아니라 외부 시스템일 수도 있다.
- **응용(Application)** 계층은 사용자가 요청한 기능을 실행한다. 핵심적인 업무 로직을 직접 구현하기보다는, 도메인 계층을 조합하여 기능을 완성한다.
- **도메인(Domain)** 계층은 시스템이 제공해야 할 **핵심 업무 규칙을 구현**하는 부분이다.
- **인프라스트럭처(Infrastructure)** 계층은 데이터베이스나 메시징 시스템처럼 외부 시스템과의 연동을 처리한다.
- 이러한 구조에서 **도메인 모델 패턴**이란, 도메인 계층을 객체 지향 기법으로 구현하는 패턴을 의미한다.

### 4.1. 개념 모델과 구현 모델

- **개념 모델**은 데이터베이스 연동 같은 기술적 고려 없이 순수하게 문제를 분석한 결과물이다. 이 때문에 실제 코드로 구현하려면, 개념 모델을 **구현 모델**로 전환하는 과정이 반드시 필요하다.
- 처음부터 완벽을 추구하기보다는 전체적인 윤곽을 파악하는 수준에서 개념 모델을 작성해야 한다. 이후 **개발을 진행하며 점진적으로 구현 모델로 발전시켜 나가는 것**이 바람직하다.

## 5. 도메인 모델 도출

- 아무리 뛰어난 개발자라 할지라도, **도메인에 대한 이해 없이** 코딩을 시작할 수는 없다.
- 도메인을 모델링하는 기본 작업은 **요구사항 분석을 통해** 모델의 핵심 구성요소와 규칙, 그리고 기능을 찾아내는 것이다.

### 5.1. 문서화

- 문서화의 주된 목적은 **지식을 공유**하는 데 있다.
- 물론 코드를 보면 실제 구현 내용을 모두 알 수 있다. 하지만 코드는 매우 상세하기 때문에, 전체적인 관점에서 소프트웨어를 이해하려면 너무 많은 시간을 투자해야 한다.

## 6. 엔티티와 밸류

- 도메인 모델의 구성 요소는 크게 **엔티티(Entity)**와 **밸류(Value)**로 구분할 수 있다.
- 도메인을 올바르게 설계하려면 이 둘의 차이를 **명확하게 이해하는 것이 매우 중요하다**.

### 6.1. 엔티티

- 엔티티의 가장 큰 특징은 **고유한 식별자**를 갖는다는 것이다.
- 두 엔티티의 **식별자가 같으면, 두 엔티티는 같다**고 판단한다.
- 엔티티의 다른 속성이 바뀌어도, 식별자는 변하지 않고 유지된다.

### 6.2. 엔티티의 식별자 생성

- 엔티티의 식별자를 생성하는 시점은 도메인의 특징과 사용하는 기술에 따라 달라진다.
- 식별자를 생성하는 대표적인 방식은 다음과 같다.
  - 특정 규칙에 따라 생성
  - UUID나 Nano ID와 같은 고유 식별자 생성기 사용
  - 값을 직접 입력
  - 일련번호 사용 (DB의 자동 증가 컬럼 등)

### 6.3. 밸류 타입

- 밸류 타입은 **개념적으로 완전한 하나를 표현**할 때 사용한다. 예를 들어 '금액'을 의미하는 Money 타입에 금액 계산 기능을 추가하는 것처럼, 관련된 기능을 함께 정의할 수 있다.
- 밸류 객체의 데이터를 변경할 때는, 기존 객체를 수정하기보다는 **수정된 데이터를 갖는 새로운 객체를 생성하는 방식을 사용**하는 것이 좋다.
- 이처럼 데이터 변경 기능을 제공하지 않는 타입을 **불변(Immutable)**이라고 하며, 밸류 타입을 불변으로 만들면 **안전한 코드를 작성**할 수 있다.

### 6.4. 엔티티 식별자와 밸류 타입

- 엔티티의 **식별자 자체를 밸류 타입으로 만들면** 의미를 더 명확하게 할 수 있다.
- 예를 들어, 단순히 문자열(String)을 사용하는 것보다 '주문 번호'라는 의미를 가진 `OrderNo`와 같은 밸류 타입을 식별자로 사용하면, 타입만으로도 그 의미를 쉽게 파악할 수 있게 된다.

### 6.5. 도메인 모델에 set 메서드 넣지 않기

- 개발자들은 습관적으로 `get/set` 메서드를 추가하는 경우가 있다.
- 하지만 **도메인 모델에 `set` 메서드를 무분별하게 추가하는 것은 좋지 않은 습관**이며, 이는 도메인의 핵심 의도를 코드에서 사라지게 만들 수 있다.
- 도메인 객체가 불완전한 상태로 사용되는 것을 막으려면, **객체 생성 시점에 생성자를 통해 필요한 모든 데이터를 전달**해야 한다.
- 만약 `set` 메서드의 기능이 꼭 필요하다면, **의도가 잘 드러나는 이름의 메서드를 만들고 공개 범위를 `private`으로 제한**하는 것이 좋다.
- 특히 앞에서 설명한 **불변 밸류 타입을 사용하면, 자연스럽게 `set` 메서드를 구현하지 않게 된다**.

## 7. 도메인 용어와 유비쿼터스 언어

- 코드를 작성할 때 도메인에서 사용하는 용어는 매우 중요하다.
- 도메인에서 사용하는 용어를 코드에 그대로 반영하면, **코드와 도메인 용어 사이의 해석 과정을 줄일 수 있다**.
- 도메인 주도 설계(DDD)에서는 이처럼 언어의 중요성을 강조하기 위해 **유비쿼터스 언어(Ubiquitous Language)** 라는 용어를 사용한다.
- **유비쿼터스 언어**란, 전문가, 관계자, 개발자 등 **모든 구성원이 도메인에 관해 이야기할 때 사용하는 공통 언어**를 의미한다.
- 중요한 점은 이 언어를 대화나 문서뿐만 아니라 **도메인 모델, 코드, 테스트 등 모든 곳에 동일하게 적용**해야 한다는 것이다.
- 한국 개발자의 경우, 적절한 영어 단어를 찾는 데 어려움을 겪을 수 있다. 하지만 **정확한 도메인 용어를 코드로 표현하려는 노력은 매우 중요하며, 충분한 시간을 투자할 가치가 있다**.
