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

## 4. 메타 애노테이션

- 커스텀 애노테이션을 정의할 때 해당 애노테이션의 동작 방식을 제어하기 위해 사용하는 **특별한 애노테이션**을 **메타 애노테이션(Meta Annotation)** 이라 한다.

### 4.1. @Retention

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    RetentionPolicy value();
}
```

```java
public enum RetentionPolicy {
    SOURCE,
    CLASS,
    RUNTIME
}
```

- 애노테이션의 **생존 기간(Lifecycle)** 을 지정한다.
  - **`RetentionPolicy.SOURCE`**: 소스 코드에만 남아있으며, 컴파일 시점(class 파일 생성 전)에 완전히 제거된다.
  - **`RetentionPolicy.CLASS`**: 컴파일 후 `.class` 파일까지는 남아있지만, 자바 프로그램이 실행(Runtime)될 때 메모리에서 제거된다. (생략 시 기본값)
  - **`RetentionPolicy.RUNTIME`**: 자바 프로그램이 실행 중일 때도 메모리에 계속 남아있어 리플렉션(Reflection)을 통해 동적으로 정보를 읽어올 수 있다. (대부분 이 설정을 사용한다.)

### 4.2. @Target

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

```java
public enum ElementType {
    TYPE,
    FIELD,
    METHOD,
    PARAMETER,
    CONSTRUCTOR,
    LOCAL_VARIABLE,
    ANNOTATION_TYPE,
    PACKAGE,
    TYPE_PARAMETER,
    TYPE_USE,
    MODULE,
    RECORD_COMPONENT;
}
```

- 애노테이션을 **적용할 수 있는 위치**를 엄격하게 지정한다.
- 배열 형태로 여러 개를 지정할 수 있으며, 주로 **`TYPE`**(클래스, 인터페이스), **`FIELD`**(멤버 변수), **`METHOD`**(메서드)를 많이 사용한다.

### 4.3. @Documented

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```

- 자바 API 문서(JavaDoc)를 생성할 때 해당 애노테이션의 메타데이터 정보가 함께 포함될지 여부를 지정한다.

### 4.4. @Inherited

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```

- **자식 클래스가 부모의 애노테이션을 상속**받을 수 있도록 지정하는 메타 애노테이션이다.
- 커스텀 애노테이션을 정의할 때 `@Inherited`를 붙이면, 해당 애노테이션이 적용된 부모 클래스를 상속받은 자식 클래스도 자동으로 같은 애노테이션을 부여받은 것으로 처리된다.
- 단, 이 기능은 **클래스 상속(extends)에서만 작동**하며, 인터페이스의 구현체(implements)에는 절대 적용되지 않는다.

#### @Inherited가 클래스 상속에만 적용되는 이유

- **클래스 상속과 인터페이스 구현의 차이**
  - 클래스 상속은 자식 클래스가 부모 클래스의 상태와 행위를 그대로 이어받으므로, 부모에 정의된 애노테이션을 자식이 상속받는 것이 논리적으로 아주 자연스럽다.
  - 반면 인터페이스는 메서드의 시그니처(규격)만을 정의할 뿐 상태나 구현을 가지지 않기 때문에, 구현체가 인터페이스의 애노테이션 메타데이터까지 상속받는다는 것은 객체 지향의 역할 분리 관점에서 잘 맞지 않는다.
- **인터페이스의 다중 구현과 다이아몬드 문제**
  - 자바에서 인터페이스는 다중 구현(Multiple Implementation)이 가능하다.
  - 만약 인터페이스의 애노테이션 상속을 허용한다면, 하나의 구현체가 서로 다른 애노테이션을 가진 여러 개의 인터페이스를 구현할 때 **어떤 애노테이션을 상속받아야 하는지 모호해지거나 충돌**하는 다이아몬드 문제가 발생할 수 있기 때문이다.

## 5. 애노테이션 상속

```java
public interface Annotation {
    boolean equals(Object obj);
    int hashCode();
    String toString();
    Class<? extends Annotation> annotationType();
}
```

- 자바의 모든 애노테이션은 내부적으로 **`java.lang.annotation.Annotation`** 인터페이스를 묵시적으로 상속받는다.
- 이 `Annotation` 인터페이스는 개발자가 직접 구현하거나 확장할 수 있는 용도가 아니라, **자바 언어 자체에서 애노테이션을 처리하기 위한 기본 기반**으로 사용된다.
  - `equals(Object obj)`: 두 애노테이션의 동일성을 비교한다.
  - `hashCode()`: 애노테이션의 해시 코드를 반환한다.
  - `toString()`: 애노테이션의 문자열 표현을 반환한다.
  - `annotationType()`: 애노테이션의 메타데이터 타입(Class)을 반환한다.
- 모든 애노테이션은 기본적으로 이 인터페이스를 확장하므로, 자바 컴파일러 내부에서 애노테이션은 **특별한 형태의 인터페이스**로 간주된다.
- 하지만 커스텀 애노테이션을 정의할 때 `implements Annotation`과 같이 **명시적으로 상속하거나 구현할 필요는 전혀 없다**.
- `@interface` 키워드를 사용하여 애노테이션을 정의하면, **자바 컴파일러가 자동으로 `Annotation` 인터페이스를 확장**하도록 내부적으로 처리해 준다.
- **주의 사항**: 애노테이션은 다른 커스텀 애노테이션이나 일반 인터페이스를 `extends` 키워드로 **직접 상속할 수 없다**. 시스템이 강제하는 `java.lang.annotation.Annotation` 인터페이스 하나만을 상속할 뿐이다.
- 따라서 일반적인 클래스나 인터페이스와 달리, 애노테이션들 사이에는 부모-자식 같은 **상속 계층 구조(Inheritance)라는 개념 자체가 아예 존재하지 않는다**.

## 6. 자바 기본 애노테이션

- 자바 언어(JDK) 자체에서 개발자의 편의와 코드 안전성을 위해 **기본적으로 제공하는 애노테이션**들도 존재한다.

### 6.1. @Override

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

- 부모 클래스나 인터페이스의 **메서드 재정의(Overriding)가 정확하게 잘 되었는지 컴파일러가 체크**하는 데 사용한다.
- 이 애노테이션을 붙이면 오타가 있거나 파라미터가 달라 재정의가 제대로 되지 않았을 때 **컴파일을 통과하지 못하게 막아준다**. 개발자의 실수를 미연에 완벽히 방지해 주므로 **사용을 강하게 권장**한다.
- **`RetentionPolicy.SOURCE`**: `@Override`는 오직 컴파일 시점에만 문법 체크 용도로 사용되고 런타임에는 전혀 필요하지 않으므로, 컴파일 이후에는 완전히 제거되도록 설정되어 있다.

### 6.2. @Deprecated

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, MODULE, PARAMETER, TYPE})
public @interface Deprecated {
    String since() default "";
    boolean forRemoval() default false;
}
```

- 해당 요소(클래스, 메서드 등)가 **더 이상 사용되지 않으며 사용을 권장하지 않음**을 나타내는 애노테이션이다.
- **사용을 권장하지 않는 주요 이유**:
  - 해당 요소를 계속 사용할 경우 시스템 오류가 발생할 가능성이 있을 때
  - 하위 호환성이 깨지거나 향후 버전에서 아예 제거될 예정일 때
  - 성능이나 구조가 더 나은 최신 대체 기술로 변경되었을 때
- **주요 속성**:
  - **`since`**: 해당 기능이 더 이상 사용되지 않게 된(Deprecated 처리된) 기준 버전 정보를 명시한다.
  - **`forRemoval`**: `true`로 설정되면 미래 버전에 해당 코드가 확실히 제거될 예정임을 나타낸다.
- **IDE 경고 수준**:
  - 속성 없이 `@Deprecated`만 붙은 코드를 호출하면, IDE에서 일반적인 경고(취소선 등)를 나타낸다.
  - `@Deprecated(forRemoval = true)`로 설정된 코드를 호출하면, IDE가 빨간색으로 **심각한 경고**를 표시하여 위험성을 알린다.
  - 단, 컴파일 시점에 경고만 발생할 뿐 **프로그램 자체는 정상적으로 컴파일되고 작동**한다.

### 6.3. @SuppressWarnings

```java
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```

- 이름 그대로 컴파일러가 띄우는 **경고를 강제로 억제(무시)** 하는 애노테이션이다.
- 자바 컴파일러가 특정 코드에 대해 위험하다고 경고하지만, 개발자가 해당 코드의 동작과 문제를 확실히 인지하고 있고 안전하다고 판단하여 **더 이상 불필요한 경고창을 띄우지 말라고 지시**할 때 사용한다.
- `@SuppressWarnings`에 문자열 배열 형태로 전달할 수 있는 **대표적인 억제 속성 값**들은 다음과 같다.
  - **`all`**: 발생하는 모든 종류의 경고를 일괄 억제한다.
  - **`deprecation`**: 사용이 권장되지 않는(`@Deprecated`) 코드를 사용할 때 발생하는 경고를 억제한다.
  - **`unchecked`**: 제네릭(Generic) 타입과 관련된 타입 안정성(unchecked) 경고를 억제한다.
  - **`serial`**: `Serializable` 인터페이스를 구현할 때 `serialVersionUID` 필드를 명시적으로 선언하지 않아 발생하는 경고를 억제한다.
  - **`rawtypes`**: 제네릭 타입 파라미터가 명시되지 않은 원시(Raw) 타입을 사용할 때 발생하는 경고를 억제한다.
  - **`unused`**: 선언해 놓고 한 번도 사용되지 않은 변수, 메서드, 필드 등에 대한 경고를 억제한다.
