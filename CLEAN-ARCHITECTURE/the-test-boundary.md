# 테스트 경계

## 1. 시스템 컴포넌트인 테스트

- 아키텍처 관점에서 **모든 테스트는 동일하다**.
- TDD로 생성한 단위 테스트부터 대규모 통합 테스트까지 모두 아키텍처적으로 **동등**하다.
- **의존성 규칙**: 테스트는 세부적이며 구체적인 것이므로, 의존성은 항상 **테스트 대상이 되는 코드(안쪽)** 를 향한다.
  - 즉, 테스트는 아키텍처에서 **가장 바깥쪽 원**이다.
- 테스트는 시스템 운영에 필수적이지 않으며, 사용자가 아닌 **개발을 지원**하는 데 역할이 있다.
- 하지만 테스트는 다른 시스템 컴포넌트가 지켜야 하는 **모델**을 표현하는 엄연한 **시스템 컴포넌트**다.

## 2. 테스트를 고려한 설계

- 테스트가 시스템 설계와 통합되지 않으면 **깨지기 쉬운 테스트 문제(Fragile Test Problem)** 가 발생한다.
- **결합(Coupling)** 의 문제: 시스템의 사소한 변경이 수천 개의 테스트 실패로 이어지면, 개발자는 변경을 주저하게 되고 시스템은 **뻣뻣**해진다.
- 해결책은 **"변동성이 있는 것에 의존하지 말라"** 는 원칙에 따라 처음부터 테스트를 고려해 설계해야 한다.

## 3. 테스트 API

- 테스트가 업무 규칙을 검증할 때 사용할 수 있는 **특화된 API**를 만든다.
- 이 API는 보안 제약사항을 무시하거나 시스템을 특정 상태로 강제하는 **강력한 힘**을 가진다.
- 테스트 API의 목표는 테스트 구조를 애플리케이션 구조로부터 **결합 분리(Decoupling)** 하는 것이다.

## 4. 구조적 결합

- **구조적 결합은 테스트 결합 중 가장 강력하고 은밀한 유형이다**.
- 상용 코드의 내부 구조가 바뀌면 다수의 테스트가 함께 변경되어야 한다.
- 그 결과 테스트는 깨지기 쉬워지고, 상용 코드의 리팩토링을 방해한다.
- 테스트 API는 상용 코드의 내부 구조를 감추는 완충 지점 역할을 한다.
  - 내부 구조가 변경되더라도 테스트는 영향을 받지 않도록 만든다.
- 시간이 지날수록 테스트는 더 구체적이고 특화되고, 상용 코드는 더 추상적이고 범용적으로 변한다.
- **상용 코드와 테스트가 서로 독립적으로 진화할 수 있어야 한다**.

### 4.1. 나쁜 예: 구조적 결합이 강한 설계

```java
// [상용 코드]
public class MemberService {
    public void saveToDatabase(String name, String email) {
        // DB 저장 로직...
    }
}

// [테스트 코드] 내부 메서드명에 직접 의존
@Test
public void 회원가입_테스트() {
    MemberService service = new MemberService();
    service.saveToDatabase("홍길동", "hong@test.com"); // 메서드명 변경 시 테스트 파손
    assertTrue(service.findAll().contains("홍길동"));
}
```

### 4.2. 좋은 예: 테스트 API를 도입한 설계

```java
// 1. [테스트 API] 상용 코드의 구조를 숨기는 방어막
public class SystemTestApi {
    private MemberService memberService = new MemberService();

    public void 회원_등록_수행(String name, String email) {
        // 내부 메서드명이 바뀌어도 여기만 수정하면 됨
        memberService.saveToDatabase(name, email);
    }

    public boolean 회원_존재_확인(String name) {
        return memberService.exists(name);
    }
}

// 2. [실제 테스트] '어떻게'가 아닌 '무엇'을 하는지에 집중
@Test
public void 개선된_회원가입_테스트() {
    SystemTestApi api = new SystemTestApi();
    api.회원_등록_수행("홍길동", "hong@test.com");
    assertTrue(api.회원_존재_확인("홍길동"));
}
```

## 5. 보안

- **테스트 API는 강력한 기능을 가지므로 운영 환경에 그대로 노출되면 위험**하다.
- 따라서 테스트 API와 위험한 구현부는 **독립적으로 분리 배포**해야 한다.
- 테스트 전용 코드와 운영 코드를 명확히 구분하는 설계가 필요하다.
