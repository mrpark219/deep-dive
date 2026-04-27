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
