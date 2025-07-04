# 냄새와 휴리스틱

## 1. 주석

### 1.1 부적절한 정보

- 다른 시스템(Jira 티켓 ID, 소스 코드 히스토리 등)에 저장할 정보는 **주석으로 작성하는 것이 적절하지 않다**.
- **주석은 코드와 설계에 대한 기술적인 설명을 부연하는 수단이다**.

### 1.2 쓸모없는 주석

- **오래되었거나 현재 코드와 맞지 않는 주석은 의미가 없다**.
- 향후 쓸모없어질 주석이라면 **애초에 달지 않는 것이 낫다**.

### 1.3 중복된 주석

- **코드만으로 충분한 설명을 다시 반복하는 주석은 중복이다**.
- **주석은 코드로 표현할 수 없는 맥락이나 배경 정보를 설명할 때만 사용해야 한다**.

### 1.4 성의 없는 주석

- 주석을 쓸 거라면 **정확하고 간결하게** 작성해야 한다.
- **성의 없는 주석은 오히려 혼란을 초래한다**.

### 1.5 주석 처리된 코드

- 코드 중간에 **주석 처리된 코드가 많으면 가독성을 해친다**.
- 얼마나 오래되었고, 현재 시스템에 어떤 영향을 주는지 **판단할 수 없기 때문에 삭제를 주저하게 된다**.
- 하지만 **그렇기 때문에 더욱더 모듈을 오염시키고 유지보수를 어렵게 만든다**.
- **주석 처리된 코드는 발견 즉시 삭제하는 것이 원칙이다**.
- **버전 관리 시스템이 변경 이력을 보존하므로 필요한 경우 언제든 복원할 수 있다**.

## 2. 환경

### 2.1 여러 단계로 빌드해야 한다

- **빌드는 반드시 한 단계로 끝나야 한다**.
- 소스 코드 관리 시스템에서 이것저것 **따로 체크아웃할 필요가 없어야 한다**.
- **불필요한 명령어, 복잡한 스크립트 실행 없이 전체를 빌드할 수 있어야 한다**.
- **한 명령으로 전체를 체크아웃하고, 한 명령으로 전체를 빌드할 수 있어야 한다**.

### 2.2 여러 단계로 테스트해야 한다

- **모든 단위 테스트는 단 한 줄의 명령으로 실행 가능해야 한다**.
- IDE에서 **버튼 하나로 전체 테스트가 돌아가는 환경이 이상적이다**.
- **빠르고, 쉽고, 명확한 방법으로 전체 테스트를 실행할 수 있어야 한다**.
- 이는 **소프트웨어 품질 유지에 있어 근본적이고 필수적인 요소다**.

## 3. 함수

### 3.1 너무 많은 인수

- **함수의 인수는 작을수록 좋다**.
- 인수가 **0개일 때 가장 이상적이다**.
- **넷 이상이라면 함수 자체의 설계가 의심스럽다**.
- 이럴 땐 **인수 객체로 묶거나 클래스로 추출해 재설계하는 것이 바람직하다**.

### 3.2 출력 인수

- **출력 인수는 직관에 반하는 설계다**.
- 일반적으로 **인수는 입력값이라는 인식이 강하기 때문에**, 출력 인수를 사용하면 **혼란을 초래한다**.
- 어떤 상태를 변경해야 한다면, **함수가 속한 객체의 내부 상태를 변경하거나 명시적으로 반환값으로 처리해야 한다**.

### 3.3 플래그 인수

- **boolean 같은 플래그 인수는 함수가 여러 기능을 한다는 신호다**.
- 하나의 함수가 두 가지 이상의 역할을 하는 것은 **SRP 위반이며 유지보수성을 떨어뜨린다**.
- **플래그 인수를 없애고, 목적별로 함수를 분리하는 것이 좋다**.

### 3.4 죽은 함수

- **아무도 호출하지 않는 함수는 과감히 삭제해야 한다**.
- 낭비를 줄이고 명확성을 높이기 위해 불필요한 코드는 제거한다.

## 4. 일반

### 4.1 한 소스 파일에 여러 언어를 사용한다

- JSP 파일은 HTML, 자바, 태그 라이브러리 구문 등을 포함한다.
- 이상적으로는 **소스 파일 하나에 언어 하나만 사용하는 방식이 좋다**.
- 각별한 노력을 기울여 **소스 파일에서 언어 수와 범위를 최대한 줄이도록 애써야 한다**.

### 4.2 당연한 동작을 구현하지 않는다

- 최소 놀람의 원칙(The Principle of Least Surprise)에 따라 **함수나 클래스는 다른 프로그래머가 당연하게 여길 만한 동작을 제공해야 한다**.
- 그렇지 않으면 **함수 이름만 보고 기능을 직관적으로 예측하기 어려워진다**.

### 4.3 경계를 올바로 처리하지 않는다

- **코드는 올바로 동작해야 한다**. 또한, 올바른 동작이 아주 복잡하다는 사실을 간과해서는 안 된다.
- **모든 경계 조건, 구석진 경우, 기벽, 예외는 철저히 다뤄야 한다**.
- 직관에 의존하지 말고 **테스트 케이스를 작성하여 모든 경계 조건을 확인해야 한다**.

### 4.4 안전 절차 무시

- **안전 절차를 무시하면 위험하다**.
- **컴파일러 경고 일부를 끄면 빌드는 쉬워질지 모르지만, 끝없는 디버깅에 시달릴 수 있다**.

### 4.5 중복

- **중복은 제거해야 한다**.
- 데이비드 토머스와 앤디 헌트는 이를 **DRY(Don’t Repeat Yourself)** 원칙이라 부른다.
- 켄트 벡은 이를 **Once, and only once**라고 했다.
- **코드에서 중복을 발견할 때마다 추상화할 기회로 간주하라**.
- **중복된 코드는 하위 루틴이나 클래스 분리로 제거한다**.
- **추상화로 중복을 정리하면 설계 언어의 어휘가 늘어난다**.

#### 중복의 주요 유형과 대응 방식

- **복사-붙여넣기 중복**: 간단한 함수로 치환한다.
- **조건문 반복**: 여러 모듈에서 동일한 `if` / `switch` 조건을 반복한다면 **다형성으로 대체한다**.
- **유사한 알고리즘 중복**: 알고리즘은 유사하나 코드가 다를 경우, **TEMPLATE METHOD** 또는 **STRATEGY 패턴**을 적용한다.

### 4.6 추상화 수준이 올바르지 못하다

- **추상화는 저차원 상세 개념에서 고차원 일반 개념을 분리하는 것이다**.
- 모든 **저차원 개념은 파생 클래스에**, 모든 **고차원 개념은 기초 클래스에 넣어야 한다**.
- **세부 구현과 관련된 상수, 변수, 유틸리티 함수는 기초 클래스에 넣으면 안 된다**.
- **기초 클래스는 구현 세부사항에 무지해야 한다**.

### 4.7 기초 클래스가 파생 클래스에 의존한다

- **기초 클래스와 파생 클래스를 구분하는 이유는 고차원 개념과 저차원 개념을 분리해 독립성을 보장하기 위해서다**.
- 따라서 **기초 클래스는 파생 클래스를 전혀 몰라야 한다**.
- 간혹 **파생 클래스의 수가 고정되어 있으면**, **기초 클래스에 파생 클래스 선택 코드가 들어갈 수 있다**.
- 대표적인 예시로 **FSM(Finite State Machine)** 구조가 있다.
  - FSM: 입력에 따라 현재 상태에서 다음 상태로 전이되는 시스템이다. 미리 정의된 대로 입력에 따라 동작하는 Machine을 의미한다.

### 4.8 과도한 정보

- **잘 정의된 모듈은 작은 인터페이스로도 많은 동작이 가능하다**.
- 부실하게 정의된 모듈은 인터페이스가 구질구질하며, 결합도가 높고 사용이 어렵다.
- **클래스나 모듈은 외부에 노출하는 함수를 최소화해야 한다**.
- 클래스가 제공하는 메서드 수는 작을수록 좋고, 함수가 접근하는 변수 수도 작을수록 좋다.
- 자료, 유틸리티 함수, 상수, 임시 변수는 숨기는 것이 바람직하다.
- **메서드와 인스턴스 변수가 넘쳐나는 클래스는 피해야 한다**.

### 4.9 죽은 코드

- **죽은 코드란 실행되지 않는 코드를 말한다.**
- 예: 불가능한 조건의 if 문, throw 문이 없는 try-catch 구문
- 죽은 코드는 시간이 지나면 설계와 어긋나 악취를 풍기기 시작한다.
- **죽은 코드를 발견하면 시스템에서 제거한다.**

### 4.10 수직 분리

- **변수와 함수는 사용되는 위치에 가깝게 정의한다**.
- 지역 변수는 처음으로 사용하기 직전에 선언하며 수직으로 가까운 곳에 위치해야 한다.
- **비공개 함수는 처음 호출한 직후에 정의한다.**
- 비공개 함수는 처음으로 호출되는 위치를 찾은 후 조금만 아래로 내려가면 찾을 수 있어야 한다.

### 4.11 일관성 부족

- **어떤 개념을 특정 방식으로 구현했다면 유사한 개념도 같은 방식으로 구현한다**.
- 표기법은 신중하게 선택하며, **선택한 표기법은 일관되게 따른다**.

### 4.12 잡동사니

- **비어 있는 생성자, 사용되지 않는 변수, 호출되지 않는 함수, 정보 없는 주석은 제거해야 한다**.
- 이런 요소는 코드를 혼란스럽게 만든다.

### 4.13 인위적 결합

- **서로 무관한 개념을 인위적으로 결합하지 않는다**.
- 예를 들어 `enum`, 범용 `static` 함수는 특정 클래스에 속할 필요가 없다.
- **함수, 상수, 변수를 선언할 때는 올바른 위치를 신중히 고민해야 한다**.

### 4.14 기능 욕심

- **클래스 메서드는 자기 클래스 내부에 집중해야 한다**.
- **다른 클래스의 데이터를 조작하려고 들면, 관심사의 범위를 넘는 것이다**.

### 4.15 선택자 인수

- **함수 호출 시 false 같은 선택자 인수는 코드 가독성을 떨어뜨린다**.
- 선택자 인수는 복잡한 기능을 하나의 함수에 억지로 담는 결과다.
- **작은 함수 여럿으로 나누는 편이 더 명확하다**.

### 4.16 모호한 의도

- 코드를 짤 때는 **의도를 최대한 분명히 밝혀야 한다**.
- 행을 바꾸지 않고 표현한 수식이나 헝가리식 표기법처럼 **가독성을 해치는 기법은 피해야 한다**.
- **의도를 명확히 표현하면 코드를 이해하고 유지보수하기 쉬워진다**.

### 4.17 잘못 지운 책임

- **소프트웨어 개발에서 코드의 위치는 매우 중요하다**.
- 최소 놀람의 원칙에 따라 **코드는 독자가 자연스럽게 기대하는 위치에 있어야 한다**.
- **함수 이름을 기준으로 코드 위치를 결정**하면 의도를 더 명확히 드러낼 수 있다.

### 4.18 부적절한 static 함수

- 일반적으로 **static 함수보다 인스턴스 함수가 더 유연하다**.
- 조금이라도 의심스러우면 static 대신 인스턴스 함수로 정의한다.
- static 함수로 정의하려면 **재정의할 가능성이 전혀 없는지 신중히 고려해야 한다**.

### 4.19 서술적 변수

- 프로그램 가독성을 높이는 가장 좋은 방법 중 하나는 **계산을 여러 단계로 나누고 서술적인 변수 이름을 사용하는 것이다**.
- **중간 결과에 의미 있는 이름을 붙이면 코드 이해가 쉬워진다**.
- **서술적인 변수는 많을수록 좋다**는 점을 기억하자.

### 4.20 이름과 기능이 일치하는 함수

- 함수 이름이 기능을 설명하지 못한다면 **코드의 의도를 이해하기 위해 구현을 들여다봐야 한다**.
- **이름이 기능을 정확히 설명하도록 바꾸거나**, 이름에 맞게 **기능을 조정해 일치시켜야 한다**.

### 4.21 알고리즘을 이해하라

- 대다수 괴상한 코드는 사람들이 **알고리즘을 충분히 이해하지 않은 채 코드를 구현한 탓**이다.
- 구현이 끝났다고 선언하기 전에 **함수가 어떻게 동작하는지 반드시 이해하고 있어야 한다**.
- 테스트를 모두 통과한다고 해서 알고리즘이 정확하다고 단정할 수 없다.

### 4.22 논리적 의존성은 물리적으로 드러내라

- **논리적으로 의존하는 모듈은 실제로도 의존해야 한다**.
- 상대 모듈에 대해 뭔가를 암묵적으로 가정해서는 안 된다.
- **필요한 정보는 명시적으로 요청**하는 방식이 안전하다.

### 4.23 If/Else 혹은 Switch/Case 문보다 다형성을 사용하라

- 대부분의 switch 문은 **당장의 편의 때문에 사용되며, 구조적으로 바람직하지 않다**.
- 유형(type)은 자주 바뀌지 않지만, 함수는 자주 바뀐다. 따라서 **switch 문은 다형성으로 대체하는 것이 좋다**.
- **동일한 switch 문이 여러 곳에 중복된다면 반드시 다형성으로 리팩터링해야 한다**.

### 4.24 표준 표기법을 따르라

- **팀은 업계 표준에 기반한 내부 구현 표준을 정의하고 따라야 한다**.
- 표기법은 클래스, 메서드, 변수의 이름, 들여쓰기, 괄호 스타일 등을 포함한다.
- **일관성은 협업에서 매우 중요한 요소**다.

### 4.25 매직 숫자는 명명된 상수로 교체하라

- 코드에 **의미 없는 숫자나 토큰을 직접 사용하는 것은 피해야 한다**.
- **모든 매직 숫자나 값은 의미 있는 상수로 추출해서 이름을 부여해야 한다**.
- 이렇게 하면 코드의 **의도와 의미가 명확해지고**, **유지보수도 쉬워진다**.

### 4.26 정확하라

- **코드에서 무언가를 결정할 때는 명확하고 구체적으로 판단해야 한다**.
- **결정의 이유와 예외 처리 방식이 코드에 명확히 드러나야 한다**.
- 예를 들어, 함수가 null을 반환할 수 있다면 **반드시 null 여부를 점검**해야 한다.
- **모호성과 부정확성은 게으름이나 의견 충돌의 산물이며, 반드시 제거해야 한다**.

### 4.27 관례보다 구조를 사용하라

- **설계 제약은 관례보다 구조로 강제하는 것이 좋다**.
- 명명 규칙만으로 일관성을 유지하는 것보다, **컴파일 타임에 검증 가능한 구조로 제약하는 것이 더 안전하다**.
- 예: `enum + switch` 조합보다 **추상 메서드를 가진 기초 클래스와 파생 클래스의 구조가 더 바람직하다**.

### 4.28 조건을 캡슐화하라

- **복잡한 조건문은 별도의 함수로 분리하여 의미를 드러내야 한다**.
- `if (user.age > 18 && user.verified)` 대신, `if (canAccessContent(user))` 와 같이 표현하는 편이 명확하다.
- **조건문의 의미를 명확히 하면 코드의 가독성과 유지보수성이 높아진다**.

### 4.29 부정 조건을 피해라

- `if (!isNotReady())` 와 같은 **이중 부정은 이해를 방해한다**.
- 가능하면 **긍정 조건으로 표현**하여 코드를 더 직관적으로 만든다.
- **부정보다 긍정이 읽고 쓰기 쉽다**.

### 4.30 함수는 한 가지만 해야 한다

- 함수가 여러 일을 처리하면 **하나의 책임만 수행하도록 나누어야 한다**.
- 예: 데이터를 조회하고, 로깅하고, 포맷팅까지 하는 함수는 **하나의 책임만 갖도록 분리해야 한다**.
- **한 함수는 하나의 추상화 수준에 집중해야 한다**.

### 4.31 숨겨진 시간적인 결합

- **함수가 특정 순서로 호출되어야 한다면, 그 순서를 코드에 명확히 드러내야 한다**.
- 순서를 바꾸면 오작동하는 코드라면, 함수 간의 **의존성과 흐름을 인수나 반환값으로 명시**해야 한다.
- **시간적 결합이 필요한 경우에도 숨기지 말고 구조적으로 표현해야 한다**.

#### ❌ 나쁜 예시

```java
order.initialize();
order.calculateTotal();
order.printInvoice();
```

- 함수 간 순서 의존이 있지만, **표현이 불명확하다**.

#### ✅ 좋은 예시

```java
Total total = order.calculateTotal();
Invoice invoice = order.createInvoice(total);
printer.print(invoice);
```

- **각 단계의 결과가 다음 단계에 명시적으로 전달되므로 실행 순서가 분명하다**.

### 4.32 일관성을 유지하라

- **코드 구조를 잡을 때는 일관된 이유와 목적이 있어야 한다**.
- 그리고 **그 이유는 코드의 구조와 배치로 분명히 드러나야 한다**.
- **구조가 일관되지 않으면 다른 사람이 마음대로 바꾸어도 된다고 오해할 수 있다**.
- 예를 들어, **다른 클래스 안에 정의된 `public` 클래스는 그 구조가 일관되지 않아 혼란을 준다**.
  - `public` 클래스는 일반적으로 **패키지 최상위 수준에서 정의하는 것이 관례**다.

### 4.33 경계 조건을 캡슐화하라

- **경계 조건은 실수하기 쉽기 때문에 반복적으로 쓰지 말고, 한 곳에서 처리하도록 캡슐화해야 한다**.
- **중복된 경계 조건은 버그의 원인이 된다**. 따라서 **의미 있는 함수로 추출해 재사용하는 것이 안전하고 가독성도 좋다**.

#### ❌ 나쁜 예시

```java
if (index >= 0 && index < list.size()) {
    return list.get(index);
}
```

#### ✅ 좋은 예시

```java
public boolean isValidIndex(List<?> list, int index) {
    return index >= 0 && index < list.size();
}

// 사용
if (isValidIndex(list, index)) {
    return list.get(index);
}
```

### 4.34 함수는 추상화 수준을 한 단계만 내려가야 한다

- 하나의 함수 안에서는 **코드 한 줄 한 줄이 비슷한 수준의 역할**을 해야 한다.
- 함수 이름이 어떤 일을 하는지 말해준다면, 그 함수 내부 코드는 **그 일을 실제로 어떻게 하는지를 보여주는 정도**여야 한다.
- 예를 들어 "사용자 등록"이라는 함수라면, 내부에서는 "이메일 유효성 검사", "비밀번호 암호화", "DB 저장" 같은 일을 해야지, **SQL 쿼리나 문자열 처리 같은 세부 구현까지 직접 하면 추상화 수준이 맞지 않는다**.
- 이런 식으로 **함수 내부 구현이 너무 구체적이면**, 나중에 코드를 이해하거나 수정할 때 **복잡하고 어려워진다**.
- 추상화 수준을 잘 맞추면, **코드를 읽는 사람이 함수의 흐름을 자연스럽게 따라갈 수 있고**, **중요한 개념이 더 잘 드러난다**.

### 4.35 설정 정보는 최상위 단계에 둬라

- 설정에 사용되는 상수나 기본값은 **코드의 가장 바깥, 즉 고차원 수준에서 정의해야 한다**.
- **하위 함수에 직접 상수를 박아 넣으면**, 나중에 설정을 바꾸기 어렵고, 코드 전체 흐름을 이해하기 힘들어진다.
- 설정 값은 고차원 함수에서 정의한 후, **인수로 하위 함수에 전달하는 방식이 좋다**.
- 그래야 설정 변경이 쉬워지고, **코드의 유연성과 가독성도 좋아진다**.

### 4.36 추이적 탐색을 피하라

- **한 모듈은 자신이 직접 사용하는 모듈만 알아야 한다**.
  - 예를 들어 A가 B를 사용하고, B가 C를 사용하더라도, **A가 C까지 알아야 할 이유는 없다**.
- 이런 구조를 피하는 원칙을 **디미터의 법칙(Law of Demeter)** 혹은 **부끄럼 타는 코드 작성(Writing Shy Code)** 이라 부른다.
- 예를 들어 `a.getB().getC()`와 같은 코드는 문제를 일으킨다.
  - **중간 객체가 변경되면 호출하는 쪽(A)도 영향을 받기 때문에 유연성이 떨어지기 때문이다**.
- **내가 사용하는 객체가 필요한 기능을 직접 제공해야 하며, 내가 직접 객체 그래프를 탐색하지 않도록 설계하는 것이 이상적이다**.

## 5. 자바

### 5.1 긴 import 목록을 피하고 와일드카드를 사용하라

- **같은 패키지에서 여러 클래스를 사용한다면 `*` 와일드카드를 사용해야 한다.**
- `import java.util.*`처럼 작성하면 다수 클래스를 포함할 때 더 간결하다.
- **명시적으로 모든 클래스를 `import`하면 코드를 읽기 어렵고 유지보수에 부담을 준다.**
- 명시적 `import`는 클래스 존재를 강하게 요구하지만, 와일드카드는 그렇지 않다.
- 물론 이름 충돌 가능성은 있지만, **대부분의 IDE는 이를 감지하고 해결을 돕는 기능을 제공한다.**
- IDE 기능을 활용해 와일드카드 `import`를 명시적 `import`로 손쉽게 변환할 수 있다.
- 테스트 코드나 스텁에서 클래스를 명시적으로 `import`해야 할 때도 있지만, **이제는 드문 경우이다.**

### 5.2 상수는 상속하지 않는다

- 상수를 상속 계층 맨 위에 정의해두고 하위 클래스에서 사용하는 방식은 **잘못된 설계**다.
- 이는 **언어의 범위 규칙을 무시하는 행위**다.
- 상수는 상속이 아닌 **`static import`를 통해 직접 사용하는 것이 명확하고 안전하다**.

### 5.3 상수 대 Enum

- 자바 5부터는 `enum`이 도입되어 **상수 집합을 보다 명확하게 표현할 수 있다**.
- 과거처럼 `public static final int` 방식으로 상수를 선언하는 대신, **`enum`을 사용해 의미를 부여하고 타입 안정성까지 확보한다**.
- `enum`은 **필드와 메서드도 가질 수 있어 더 풍부한 표현이 가능하다**.

## 6. 이름

### 6.1 서술적인 이름을 사용하라

- **이름은 성급하게 짓지 않고, 신중하게 고른다**.
- **소프트웨어가 진화함에 따라 의미도 변하므로, 이름이 여전히 적절한지 자주 되돌아본다**.
- **좋은 이름은 추가 설명이 필요 없는 코드를 만든다**.

### 6.2 적절한 추상화 수준에서 이름을 선택하라

- **구현 세부사항을 드러내는 이름은 피하고**, 해당 코드의 추상화 수준에 맞는 이름을 선택한다.

### 6.3 가능하다면 표준 명명법을 사용하라

- **관례를 따르는 이름은 이해하기 쉽다**.
- 팀 내에서는 **프로젝트의 유비쿼터스 언어**처럼 통일된 명명 규칙을 공유하는 것이 좋다.

### 6.4 명확한 이름

- **함수나 변수의 목적을 분명히 드러내는 이름을 사용한다**.

### 6.5 긴 범위는 긴 이름을 사용하라

- **이름 길이는 사용 범위에 비례해야 한다**.
- 범위가 작으면 짧은 이름도 괜찮지만, **범위가 클수록 의미 있는 긴 이름이 필요하다**.

### 6.6 인코딩을 피하라

- **이름에 타입이나 범위를 암시하는 접두어를 넣지 않는다**.
- 헝가리안 표기법처럼 **중복되거나 혼란을 주는 방식은 피한다**.

### 6.7 이름으로 부수 효과를 설명하라

- **이름은 함수나 클래스가 하는 모든 일을 담아야 한다**.
- **부수 효과를 숨기지 말고 명확히 드러낸다**.

## 7. 테스트

### 7.1 불충분한 테스트

- 테스트는 **깨질 가능성이 있는 모든 조건과 계산을 검증해야 한다**.
- 검증이 빠진 테스트는 불완전하다.

### 7.2 커버리지 도구를 사용하라

- 커버리지 도구는 테스트가 누락된 부분을 알려준다.
- **불충분한 테스트 범위를 쉽게 식별할 수 있다**.

### 7.3 사소한 테스트를 건너뛰지 마라

- 사소한 테스트라도 구현 비용보다 문서로서의 가치가 크다.
- 작은 테스트일수록 작성은 쉽고 효과는 크다.

### 7.4 무시한 테스트는 모호함을 뜻한다

- @Ignore나 주석 처리된 테스트는 **요구사항이 명확하지 않다는 신호다**.
- 불확실한 부분은 명확해질 때까지 기록하거나 명시적으로 남겨둔다.

### 7.5 경계 조건을 테스트하라

- 경계는 가장 흔히 실수하는 부분이다.
- 중앙값보다도 **경계값을 더 철저히 테스트해야 한다**.

### 7.6 버그 주변은 철저히 테스트하라

- **버그는 모여서 발생하는 경향이 있다**.
- 버그가 나타난 함수는 주변까지 포함해 집중적으로 테스트한다.

### 7.7 실패 패턴을 살펴라

- 테스트 실패에는 패턴이 있다.
- 잘 정렬된 테스트는 문제의 근원을 빠르게 파악하는 데 도움이 된다.

### 7.8 테스트 커버리지 패턴을 살펴라

- 성공한 테스트가 실행하지 않은 코드 중에 문제가 있는 경우가 많다.
- 커버리지를 통해 실패 원인을 좁혀갈 수 있다.

### 7.9 테스트는 빨라야 한다

- **느린 테스트 케이스는 실행하지 않게 된다**.
- 테스트가 빠를수록 자주 실행되고 유지된다.

## 8. 마무리

- **휴리스틱을 안다고 해서 훌륭한 개발자가 되는 것은 아니다**.
- **전문성과 장인정신은 올바른 가치관에서 시작된다**.
- 그 가치를 지키기 위한 **규율과 절제**가 필요하다.
