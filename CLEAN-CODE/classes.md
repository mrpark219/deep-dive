# 클래스

## 1. 클래스 체계

- 클래스를 정의하는 표준 자바 관례에 따르면 가장 먼저 변수 목록이 나온다.
- 정적 공개(`static public`) 상수가 있다면 맨 처음에 나온다.
- 다음으로 정적 비공개(`static private`) 변수가 나오며 이어서 비공개 인스턴스 변수가 나온다.
- **공개 변수가 필요한 경우는 거의 없다**.
- 변수 목록 다음에는 공개 함수가 나온다. 비공개 함수는 자신을 호출하는 공개 함수 직후에 넣는다. **즉 추상화 단계가 순서대로 내려간다**.

### 1.1 캡슐화

- 변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 **반드시 숨겨야 한다는 법칙도 없다**.
- 때로는 변수나 유틸리티 함수를 `protected`로 선언해 테스트 코드에 접근을 허용하기도 한다.
- **캡슐화를 풀어주는 결정은 언제나 최후의 수단이다**.

## 2. 클래스는 작아야 한다

- 클래스를 만들 때 첫 번째 규칙은 **크기**다.
- **클래스는 작아야 한다**.
- 클래스를 설계할 때나 함수를 설계할 때나 **작게가 기본 규칙**이다.
- 클래스가 작다는 것은 클래스가 맡은 **책임이 작다**는 것이다.
- 클래스의 이름은 해당 클래스의 책임을 **명확히 기술**해야 한다.
- 클래스 이름이 **모호하다면** 클래스를 **분리해야 할 때**가 왔다는 뜻이다.

### 2.1 단일 책임 원칙 (SRP: Single Responsibility Principle)

- 클래스나 모듈을 **변경할 이유가 하나, 단 하나뿐이어야 한다**.
- 책임이라는 개념을 정의하며 **적절한 클래스 크기**를 제시한다.
- 책임(변경할 이유)을 파악하면 코드를 **추상화**하기도 더 쉬워진다.
- **동작하는 코드와 깨끗한 코드는 다르다**.
- 클래스 수가 많아도 돌아가는 부품 수는 비슷하다. **작은 클래스가 많은 쪽이 더 유지보수에 좋다**.

### 2.2 응집도(Cohesion)

- 메서드가 인스턴스 변수를 **더 많이 사용할수록 응집도가 높다**.
- 모든 인스턴스 변수를 메서드마다 사용하는 클래스는 응집도가 가장 높다.
- 응집도가 높다는 말은 클래스의 메서드와 변수가 **서로 강하게 연관**되어 있다는 의미다.
- 메서드 일부만 특정 변수를 쓴다면 **클래스를 쪼개야 한다는 신호**다.

### 2.3 응집도를 유지하면 작은 클래스가 여럿이 나온다

- 큰 함수를 작은 함수 여럿으로 나누면 클래스 수도 자연스럽게 많아진다.
- 클래스가 응집력을 잃는다면 **쪼개야 한다**.

## 3. 변경하기 쉬운 클래스

- 대부분 시스템은 **지속적인 변경**이 가해진다.
- 변경이 생기면 시스템이 의도대로 동작하지 않을 위험이 생긴다.
- **기존 코드를 최소한만 수정**하고 새 기능을 추가할 수 있다면 좋은 설계다.
- **이상적인 시스템은 기존 코드를 손대지 않고 확장만 한다**.

### 3.1 변경으로부터 격리

- 상세 구현에 의존하는 코드는 **변경에 민감하다**.
- **인터페이스나 추상 클래스를 통해 구체적인 구현과 클라이언트를 분리**해야 한다.
- **결합도를 낮추면 테스트가 쉬워지고, 유연성과 재사용성이 높아진다**.
- 이는 **DIP(Dependency Inversion Principle)** 을 따르게 만든다.
- **상세한 구현이 아닌 추상화에 의존해야 한다**.

## 4. 마무리

- **클래스는 작아야 하며 단일 책임을 가져야 한다**.
- **응집도를 유지하며 결합도를 낮추는 설계**가 이상적이다.
- **변경이 용이한 설계**를 위해 추상화와 캡슐화를 적극적으로 활용해야 한다.
- 코드는 동작하는 것에 만족하지 말고, **깨끗하고 유지보수하기 쉽게 만들어야 한다**.
