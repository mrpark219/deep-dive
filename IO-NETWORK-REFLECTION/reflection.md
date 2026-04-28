# 리플렉션

## 1. 리플렉션이 필요한 이유

- 커맨드 패턴으로 만든 서블릿은 아주 유용하지만, 몇 가지 단점이 있다.

### 1.1. 하나의 클래스에 하나의 기능만 만들 수 있다

```java
public class Site1Servlet implements HttpServlet {
    @Override
    public void service(HttpRequest request, HttpResponse response) {
        response.writeBody("<h1>site1</h1>");
    }
}
```

```java
public class Site2Servlet implements HttpServlet {
    @Override
    public void service(HttpRequest request, HttpResponse response) {
        response.writeBody("<h1>site2</h1>");
    }
}
```

- 커맨드 패턴 기반의 서블릿 구조에서는 **하나의 클래스에 오직 하나의 기능(메서드)만** 만들 수 있다는 한계가 있다.
- 위 코드처럼 아주 단순한 기능 하나를 추가할 때마다 **각각 별도의 서블릿 클래스를 새로 만들고 인터페이스를 구현**해야만 한다.
- 이러한 방식은 역할이 명확히 나뉘어 복잡한 기능에서는 효과적일 수 있지만, 간단한 기능들을 여러 개 만들 때는 **클래스 개수가 기하급수적으로 늘어나 관리 부담이 매우 커진다**.

### 1.2. 새로운 클래스를 URL 경로와 매번 수동으로 매핑해야 한다

```java
servletManager.add("/", new HomeServlet());
servletManager.add("/site1", new Site1Servlet());
servletManager.add("/site2", new Site2Servlet());
servletManager.add("/search", new SearchServlet());
servletManager.add("/favicon.ico", new DiscardServlet());
```

- 새로운 서블릿 클래스를 만들 때마다 위 코드처럼 **URL 경로 문자열과 해당 서블릿 객체를 항상 수동으로 매핑(연결)** 해 주는 코드를 추가해야 한다.
- 만약 클라이언트가 요청한 **URL 경로의 이름과 완벽히 동일한 이름의 메서드**를 특정 클래스 내부에서 동적으로 찾아서 호출할 수 있다면 어떨까?
- 자바의 기능(리플렉션)을 활용하여 **클래스에 있는 메서드의 이름을 런타임에 찾아 실행**할 수 있게 만들면, 위와 같은 번거로운 수동 매핑 작업을 깔끔하게 원천 제거할 수 있다.

## 2. 클래스와 메타데이터

- 클래스가 제공하는 다양한 정보를 프로그램 실행 중에 **동적으로 분석하고 사용**하는 기능을 **리플렉션(Reflection)** 이라고 한다.
- 리플렉션을 활용하면 프로그램 실행 중에 클래스, 메서드, 필드 등에 대한 정보를 얻을 수 있으며, **새로운 객체를 생성**하거나 **메서드를 호출**하고, **필드의 값을 읽고 쓰는 작업**도 가능하다.
- 리플렉션이라는 용어는 '반사하다' 또는 '되돌아보다'라는 의미를 가진 영어 단어 **reflect**에서 유래되었다.
  - 즉, 프로그램이 **실행 중에 자신의 내부 구조를 거울처럼 들여다보고**, 필요에 따라 그 구조를 동적으로 변경하거나 조작할 수 있는 강력한 기능을 의미한다.
- 자바에서 리플렉션을 사용하면 구체적으로 다음과 같은 **4가지 핵심 정보**를 얻고 조작할 수 있다.
  - **클래스의 메타데이터**: 해당 클래스의 이름, 접근 제어자, 상속받은 부모 클래스, 구현된 인터페이스 등의 기본 정보를 확인할 수 있다.
  - **필드 정보**: 클래스 내부에 선언된 필드의 이름, 데이터 타입, 접근 제어자를 확인하고, 실행 중에 해당 필드의 값을 직접 읽거나 강제로 수정할 수 있다.
  - **메서드 정보**: 메서드의 이름, 반환(Return) 타입, 매개 변수(Parameter) 정보를 확인하고, 실행 중에 조건에 맞는 메서드를 동적으로 찾아 호출할 수 있다.
  - **생성자 정보**: 클래스 생성자의 매개 변수 타입과 개수를 확인하고, 런타임 환경에서 이 정보를 바탕으로 동적으로 새로운 객체 인스턴스를 생성할 수 있다.

### 2.1. 클래스의 메타데이터

```java
// 클래스 메타데이터 조회 방법 3가지

// 1. 클래스에서 찾기
Class<BasicData> basicDataClass1 = BasicData.class;
System.out.println("basicDataClass1 = " + basicDataClass1);

// 2. 인스턴스에서 찾기
BasicData basicInstance = new BasicData();
Class<? extends BasicData> basicDataClass2 = basicInstance.getClass();
System.out.println("basicDataClass2 = " + basicDataClass2);

// 3. 문자로 찾기
String className = "reflection.data.BasicData"; // 패키지명 포함 전체 이름 주의
Class<?> basicDataClass3 = Class.forName(className);
System.out.println("basicDataClass3 = " + basicDataClass3);
```

- 자바에서 클래스의 메타데이터는 **`Class`** 라는 이름의 클래스 객체로 표현된다.
- 이러한 `Class` 객체를 획득하는 방법은 크게 다음과 같이 **3가지**가 있다.
  - **클래스에서 찾기**: `클래스명.class`를 사용하여 정적으로 획득한다.
  - **인스턴스에서 찾기**: 이미 생성된 객체 참조를 통해 `인스턴스.getClass()` 메서드를 호출하여 획득한다.
  - **문자로 찾기**: 패키지 경로를 포함한 문자열을 이용해 `Class.forName("클래스명")`으로 런타임에 동적으로 획득한다.

#### 참고 1: 인스턴스 조회 시 Class<? extends Type> 반환 이유

```java
Parent parent = new Child();
Class<? extends Parent> parentClass = parent.getClass();
```

- 인스턴스의 `getClass()` 메서드를 통해 `Class` 객체를 획득할 때는 반환 타입에 제네릭 와일드카드인 **`? extends BasicData`** 가 사용된다.
- 그 이유는 다형성으로 인해 변수에 선언된 타입과 **실제 메모리에 생성된 인스턴스의 타입이 서로 다를 수 있기 때문**이다.
  - 위 코드처럼 `Parent` 타입의 참조 변수로 `getClass()`를 호출하더라도, 실제 생성되어 있는 인스턴스는 자식인 `Child` 타입일 수 있다.
  - 따라서 제네릭 문법에서 이러한 **자식 타입의 `Class` 객체까지 모두 안전하게 허용**하고 담을 수 있도록 `? extends Parent` 문법을 사용하는 것이다.

### 2.2. 클래스 세부 정보 및 수정자(Modifier) 조회

```java
Class<BasicData> basicData = BasicData.class;

// 기본 정보 조회
System.out.println("basicData.getName() = " + basicData.getName());
System.out.println("basicData.getSimpleName() = " + basicData.getSimpleName());
System.out.println("basicData.getPackage() = " + basicData.getPackage());

// 상속 및 구현 정보 조회
System.out.println("basicData.getSuperclass() = " + basicData.getSuperclass());
System.out.println("basicData.getInterfaces() = " + Arrays.toString(basicData.getInterfaces()));

// 클래스 타입 확인
System.out.println("basicData.isInterface() = " + basicData.isInterface());
System.out.println("basicData.isEnum() = " + basicData.isEnum());
System.out.println("basicData.isAnnotation() = " + basicData.isAnnotation());

// 수정자(Modifier) 정보 조회
int modifiers = basicData.getModifiers();
System.out.println("basicData.getModifiers() = " + modifiers);
System.out.println("isPublic = " + Modifier.isPublic(modifiers));
System.out.println("Modifier.toString() = " + Modifier.toString(modifiers));
```

- 획득한 `Class` 객체를 통해 클래스 이름, 패키지, 부모 클래스, 구현한 인터페이스, 수정자 정보 등 **다양한 세부 메타데이터**를 조회할 수 있다.
- **수정자(Modifier) 정보**는 `getModifiers()` 메서드를 통해 조회할 수 있으며, 각 수정자가 조합된 **정수(숫자)** 형태로 반환된다.
  - **접근 제어자**: `public`, `protected`, `default`(`package-private`), `private`
  - **비접근 제어자(기타 수정자)**: `static`, `final`, `abstract`, `synchronized`, `volatile` 등
- 반환된 숫자는 자바가 제공하는 **`Modifier`** 클래스의 헬퍼 메서드(`Modifier.isPublic()`, `Modifier.toString()` 등)를 사용하면 **실제 적용된 수정자 문자열이나 상태 정보로 쉽게 확인**할 수 있다.

### 2.3. 메서드 탐색

```java
Class<BasicData> helloClass = BasicData.class;

System.out.println("==== methods() ====");
Method[] methods = helloClass.getMethods();
for (Method method : methods) {
    System.out.println("method = " + method);
}

System.out.println("==== declaredMethods() ====");
Method[] declaredMethods = helloClass.getDeclaredMethods();
for (Method method : declaredMethods) {
    System.out.println("declaredMethod = " + method);
}
```

- 리플렉션을 사용하면 클래스에 정의된 메서드 정보를 배열(`Method[]`) 형태로 획득할 수 있으며, 탐색 범위에 따라 두 가지 메서드를 구분해서 사용한다.
  - **`getMethods()`**: 해당 클래스는 물론 상위 클래스에서 상속받은 메서드까지 포함하여 **모든 `public` 메서드를 반환**한다.
  - **`getDeclaredMethods()`**: 접근 제어자(`public`, `private` 등)와 관계없이 **해당 클래스에서 직접 선언된 모든 메서드를 반환**한다. (단, 부모로부터 상속받은 메서드는 포함하지 않는다.)

### 2.4. 동적 호출

```java
public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    BasicData helloInstance = new BasicData();

    // 1. 일반적인 메서드 호출 (정적)
    helloInstance.call(); // 코드를 변경하지 않는 이상 정적으로 고정됨

    // 2. 동적 메서드 호출 (리플렉션 사용)
    Class<? extends BasicData> helloClass = helloInstance.getClass();
    String methodName = "hello"; // 메서드 이름을 변수로 지정 및 변경 가능

    Method method1 = helloClass.getDeclaredMethod(methodName, String.class);
    Object returnValue = method1.invoke(helloInstance, "hi");
    System.out.println("returnValue = " + returnValue);
}
```

- **일반적인 메서드 호출 (정적 호출)**
  - `helloInstance.call()`과 같이 인스턴스의 참조를 통해 직접 메서드를 호출하는 가장 일반적인 방식이다.
  - 호출할 메서드가 소스 코드 상에 하드코딩되어 있으므로, 코드를 직접 수정하고 재컴파일하지 않는 한 다른 메서드로 변경할 수 없는 **정적(Static)** 인 상태이다.
- **리플렉션을 활용한 동적 호출**
  - 리플렉션의 `Class.getDeclaredMethod("메서드명", 파라미터타입)`을 사용하면 원하는 메서드를 문자열 변수를 통해 찾아낼 수 있다.
  - 찾은 메서드 객체에 **`Method.invoke(실행할_인스턴스, 전달할_인자)`** 를 호출하면, 런타임에 해당 인스턴스의 메서드를 실행하고 반환값을 얻을 수 있다.
  - 이 방식은 호출할 메서드 대상이 코드에 고정된 것이 아니라 변수의 값에 따라 언제든지 유연하게 변경될 수 있으므로 **동적(Dynamic) 메서 호출**이라고 부른다.

### 2.5. 필드 탐색

```java
Class<BasicData> helloClass = BasicData.class;

System.out.println("===== fields() ====");
Field[] fields = helloClass.getFields();
for (Field field : fields) {
    System.out.println("field = " + field);
}

System.out.println("==== declaredFields() ====");
Field[] declaredFields = helloClass.getDeclaredFields();
for (Field declaredField : declaredFields) {
    System.out.println("declaredField = " + declaredField);
}
```

- 리플렉션을 사용하면 클래스에 정의된 필드 정보(`Field[]`)를 획득할 수 있으며, 탐색 범위에 따라 두 가지 메서드를 구분해서 사용한다.
  - **`getFields()`**: 해당 클래스는 물론 상위 클래스에서 상속받은 필드까지 포함하여 **모든 `public` 필드를 반환**한다.
  - **`getDeclaredFields()`**: 접근 제어자(`public`, `private` 등)와 관계없이 **해당 클래스에서 직접 선언된 모든 필드를 반환**한다. (단, 부모로부터 상속받은 필드는 포함하지 않는다.)

### 2.6. 필드 값 변경

```java
public class User {
    private String name;

    public User() {
    }

    public User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

```java
User user = new User("userA");
System.out.println("기존 이름 = " + user.getName());

Class<? extends User> aClass = user.getClass();
Field nameField = aClass.getDeclaredField("name");

// private 필드에 접근 허용, private 메서드도 이렇게 호출 가능
nameField.setAccessible(true);
nameField.set(user, "userB");
System.out.println("변경된 이름 = " + user.getName());
```

- 리플렉션은 **`private` 필드나 메서드에 접근할 수 있는 특별한 기능**을 제공한다.
  - **`setAccessible(true)`**: 이 옵션을 켜면 자바의 접근 제어자를 무시하고 `private` 요소에 강제로 접근할 수 있다. (메서드를 나타내는 `Method` 객체에도 동일하게 적용하여 `private` 메서드를 호출할 수 있다.)
  - **`Field.set(인스턴스, 변경할_값)`**: 특정 인스턴스에 있는 해당 필드의 값을 원하는 값으로 동적으로 변경한다.

#### 참고: 리플렉션 사용 시 주의사항

- 리플렉션을 활용해 `private` 접근 제어자를 무시하고 값을 변경하는 것은 내부 데이터를 보호하는 **객체 지향 프로그래밍의 핵심 원칙인 캡슐화를 정면으로 위반**하는 행위이다.
- 클래스의 내부 구조나 세부 구현 사항(필드명 변경 등)이 바뀔 경우, 리플렉션에 의존하는 코드는 컴파일 타임에 오류를 잡지 못하고 런타임에 쉽게 깨지게 되어 **유지보수성에 큰 악영향**을 초래한다.
- 따라서 리플렉션은 프레임워크, 라이브러리 개발이나 테스트 환경 같은 **특수한 상황에서만 제한적으로 유용하게 사용**해야 한다.
- 일반적인 비즈니스 애플리케이션 코드에서는 무분별한 리플렉션 사용을 엄격히 지양하고, 가급적 **접근 메서드(Getter, Setter)** 를 사용하여 가독성과 안전성을 유지하는 것이 바람직하다.
