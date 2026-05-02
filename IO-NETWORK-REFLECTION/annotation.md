# 애노테이션

## 1. 애노테이션이 필요한 이유

- 기존의 리플렉션 서블릿 방식은 요청 URL과 메서드 이름이 완벽히 같아야만 동적으로 호출할 수 있다는 한계가 있다.
- 하지만 웹 개발 중 **요청 URL과 실제 처리하는 메서드의 이름을 다르게 설정**하고 싶은 경우가 많다.
  - 예를 들어 `/site1`이라는 URL 요청이 왔을 때, 내부적으로는 `page1()`이라는 더 명확하고 구체적인 이름의 메서드를 호출하고 싶을 수 있다.
  - 또한, 자바 메서드 이름으로는 지정할 수 없는 특수 기호(`/`)나, URL에서 구분자로 자주 쓰이는 대시(`-`)가 포함된 경로(`/add-member`)는 메서드 이름만으로 매핑하는 것이 아예 불가능하다.
- 이러한 문제를 해결하려면 메서드 이름 외에 **추가적인 매핑 정보**를 코드 어딘가에 적어두고 런타임에 읽을 수 있어야 한다.
- 만약 일반적인 주석(`//`)을 실행 중에 읽어서 URL 경로와 비교할 수 있다면 좋겠지만, 일반 주석은 컴파일 시점에 모두 제거되므로 불가능하다.
- 이렇게 **프로그램 실행 중에 기계가 읽어서 동적으로 활용할 수 있는 특별한 주석**의 필요성 때문에 등장한 기술이 바로 **애노테이션(Annotation)** 이다.

## 2. 애노테이션 예제 및 활용

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface SimpleMapping {
    String value();
}
```

- 커스텀 애노테이션은 클래스나 인터페이스를 선언하듯 **`@interface`** 키워드를 사용하여 생성한다.
- 위 코드에서는 내부에 `value`라는 `String` 타입의 속성(Attribute)을 가지는 `@SimpleMapping` 애노테이션을 정의했다.
- **`@Retention(RetentionPolicy.RUNTIME)`**: 애노테이션의 생존 주기를 지정하는 설정으로, 프로그램이 **실행되는 시점(Runtime)** 까지 이 애노테이션 정보가 살아남아 리플렉션으로 읽을 수 있게 해주는 필수 설정이다.

```java
@SimpleMapping(value = "/")
public void home() {
    System.out.println("TestController.home()");
}

@SimpleMapping("/site1")
public void page1() {
    System.out.println("TestController.page1()");
}
```

- 애노테이션을 실제 코드에 적용할 때는 **`@` 기호**를 붙여서 사용한다. (예: `@SimpleMapping`)
- 애노테이션 자체는 실행되는 프로그램 코드가 아니므로, 위처럼 메서드 위에 붙여두어도 프로그램의 핵심 로직(출력 등)에는 아무런 영향을 주지 않는다.
- 단지 해당 메서드에 대한 추가적인 정보(URL 경로)를 달아두고, 실행 시점에 리플렉션으로 읽어 들이기 위한 메타데이터 역할을 할 뿐이다.

```java
TestController testController = new TestController();

Class<? extends TestController> aClass = testController.getClass();
for (Method method : aClass.getDeclaredMethods()) {
    SimpleMapping simpleMapping = method.getAnnotation(SimpleMapping.class);
    if (simpleMapping != null) {
        System.out.println("[" + simpleMapping.value() + "] -> " + method);
    }
}
```

- 리플렉션이 제공하는 **`getAnnotation(애노테이션클래스.class)`** 메서드를 사용하면 특정 요소에 붙어있는 애노테이션 정보를 동적으로 찾을 수 있다.
  - `Class`, `Method`, `Field`, `Constructor` 등 반환된 리플렉션 객체들은 모두 자신에게 붙은 애노테이션을 조회할 수 있는 `getAnnotation()` 메서드를 기본적으로 제공한다.
- 위 코드에서는 `Method.getAnnotation(SimpleMapping.class)`를 호출하여 해당 메서드에 `@SimpleMapping`이 붙어있는지 확인한다. (없으면 `null` 반환)
- 애노테이션 객체를 성공적으로 찾았다면, **`simpleMapping.value()`** 를 호출하여 애노테이션을 선언할 때 괄호 안에 지정해 두었던 실제 매핑 값(`/` 또는 `/site1`)을 동적으로 조회할 수 있다.

### 2.1. 참고: 애노테이션(Annotation) 단어의 유래

- 영어 단어 Annotation은 원래 **주석** 또는 **메모**라는 의미를 가지고 있다.
- 자바의 애노테이션 역시 코드에 추가적인 정보를 주석처럼 제공한다는 점에서는 일반 주석과 동일하다.
- 하지만 단순한 텍스트인 일반 주석과 달리, 애노테이션은 컴파일러나 런타임 환경에서 기계가 해석하고 처리할 수 있는 **메타데이터(Metadata)** 를 제공한다는 결정적인 차이가 있다.
- 즉, 애노테이션이라는 이름은 코드 자체의 로직을 변경하지 않으면서, 기계가 읽을 수 있는 유용한 **특정 지시사항이나 정보(메모)를 코드에 달아놓는다**는 뜻에서 유래되었다.

## 3. 애노테이션 정의

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface AnnoElement {
    String value();

    int count() default 0;

    String[] tags() default {};

    // 일반적인 클래스 타입은 적용 불가 (컴파일 오류 발생)
    // MyLogger data();
    // Class(메타데이터) 타입은 허용됨
    Class<? extends MyLogger> annoData() default MyLogger.class;
}
```

- 커스텀 애노테이션은 **`@interface`** 키워드를 사용하여 정의한다.
- 애노테이션은 내부에 여러 속성(요소)을 가질 수 있으며, 이는 인터페이스에서 메서드를 선언하는 것과 매우 비슷한 형태로 정의한다.

### 3.1. 애노테이션 정의 규칙

#### 데이터 타입 제약

- 애노테이션의 요소로 사용할 수 있는 반환 데이터 타입은 다음과 같이 엄격하게 제한되어 있다.
  - 기본 타입 (`int`, `float`, `boolean` 등)
  - `String`
  - `Class` (클래스 메타데이터) 또는 인터페이스
  - `enum` (열거형)
  - 다른 애노테이션 타입
  - 위에서 나열한 허용된 타입들의 **배열**
- **주의**: 위에서 설명한 타입 외에는 절대 정의할 수 없다. 즉, `Member`, `User`, `MyLogger`와 같은 **일반적인 사용자 정의 참조 클래스는 요소 타입으로 사용할 수 없다**.

#### 기본값 (default)

- **`default`** 키워드를 사용하여 요소에 기본값을 지정할 수 있다.
- 애노테이션 사용 시 값을 생략하면 이 지정된 기본값이 적용된다. (예: `String value() default "기본값";`)

#### 요소 이름 및 매개변수 규칙

- 요소는 반드시 메서드 선언 형태로 정의되어야 한다.
- 괄호 `()`를 포함하되 내부에 **매개변수를 절대 가질 수 없다**.

#### 반환 타입 제약

- 요소(메서드)의 반환 타입으로 **`void`를 사용할 수 없다**.

#### 예외 선언 불가

- `throws` 키워드를 사용하여 메서드에 **예외를 선언할 수 없다**.

#### 특별한 요소 이름 (value)

- 애노테이션 내부에 **`value`**라는 이름의 요소 하나에만 값을 전달할 때는, 애노테이션 사용 시 속성 이름(`value = `)을 **생략**하고 값만 괄호 안에 깔끔하게 넣을 수 있다.

### 3.2. 애노테이션 사용

```java
@AnnoElement(value = "data", count = 10, tags = {"t1", "t2"})
public class ElementData1 {
}

@AnnoElement(value = "data", tags = "t1")
public class ElementData2 {
}

@AnnoElement("data")
public class ElementData3 {
}
```

- 애노테이션을 실제 코드에 적용할 때, 코드를 더 간결하게 작성할 수 있는 **3가지 생략 규칙**이 있다.
  - **`default` 값 생략**: 애노테이션 정의 시 `default` 값이 지정된 항목(`count` 등)은 값을 명시적으로 전달하지 않아도 자동으로 기본값이 적용되므로 할당을 생략할 수 있다. (참고: `ElementData2`, `ElementData3`)
  - **배열 `{}` 생략**: 배열 타입의 속성(`tags`)에 전달할 데이터 항목이 **단 하나**인 경우에는 중괄호 `{}`를 생략하고 바로 값을 입력할 수 있다. (참고: `ElementData2`의 `tags = "t1"`)
  - **`value` 키워드 생략**: 애노테이션에 값을 전달할 요소가 오직 **`value` 하나뿐인 경우**(나머지는 default가 적용되거나 아예 다른 요소가 없는 경우), `value = ` 키워드 자체를 생략하고 괄호 안에 값만 바로 적을 수 있다. (참고: `ElementData3`의 `"data"`)
