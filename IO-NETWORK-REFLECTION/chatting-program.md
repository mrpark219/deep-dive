# 채팅 프로그램

## 1. 요구사항

- 서버에 접속한 모든 사용자는 실시간으로 대화할 수 있어야 한다.
- 채팅 서버는 다음과 같은 5가지 **채팅 명령어**를 지원해야 한다.
- **입장 (`/join|{name}`)**: 처음 채팅 서버에 접속할 때 사용자의 이름을 입력하는 명령어이다.
- **메시지 (`/message|{내용}`)**: 채팅방에 접속한 모든 사용자에게 해당 내용의 메시지를 전달한다.
- **이름 변경 (`/change|{name}`)**: 사용자의 채팅 이름을 새롭게 변경한다.
- **전체 사용자 (`/users`)**: 현재 채팅 서버에 접속해 있는 전체 사용자 목록을 콘솔에 출력한다.
- **종료 (`/exit`)**: 채팅 서버와의 접속을 완전히 종료한다.

## 2. 클라이언트 고려 사항

- 채팅은 실시간으로 대화를 주고받아야 하므로, 기존 클라이언트 프로그램처럼 사용자의 콘솔 입력이 있을 때까지 프로그램이 무한정 대기하는 구조로는 실시간 통신이 불가능하다.
- 하나의 스레드가 사용자의 콘솔 입력을 대기하고 있으면, 그동안 서버를 통해 다른 사용자가 보낸 메시지가 도착하더라도 이를 즉시 콘솔에 출력할 수 없다.
- 사용자의 콘솔 입력을 기다리는 코드(`Scanner` 등)도 실행 흐름을 멈추는 **블로킹(Blocking)** 연산이고, 서버로부터 메시지를 읽어들이는 코드(`read()` 등) 역시 **블로킹** 연산이다.
- 따라서 원활한 실시간 채팅을 위해서는 **사용자의 콘솔 입력을 처리하는 작업**과 **서버로부터 메시지를 수신하는 작업**을 각각 **별도의 스레드로 분리**하여 동시에 실행되도록 설계해야 한다.

## 3. 서버 고려 사항

- 채팅 프로그램의 핵심 기능은 **일대다(1:N) 통신**으로, 한 명의 사용자가 보낸 메시지를 접속한 모든 사용자가 실시간으로 들을 수 있어야 한다.
- 이를 위해 서버는 특정 클라이언트로부터 메시지를 수신하면, 이를 단순히 보관하는 것이 아니라 접속된 **모든 클라이언트에게 다시 전달(Broadcasting)** 하는 역할을 수행해야 한다.
- 모든 클라이언트에게 메시지를 빠짐없이 전송하기 위해서는 서버가 현재 연결된 **모든 세션(Session)을 통합적으로 관리**할 수 있는 구조가 필수적이다.
- 세션 매니저와 같은 별도의 관리 객체를 통해 모든 세션의 정보를 유지해야만, 새로운 메시지가 발생했을 때 전체 세션 리스트를 순회하며 메시지를 성공적으로 배포할 수 있다.

## 4. Null Object Pattern

```java
interface Coupon {
    int applyDiscount(int price);
}

class FixedCoupon implements Coupon {
    private final int discountAmount;

    public FixedCoupon(int discountAmount) {
        this.discountAmount = discountAmount;
    }

    @Override
    public int applyDiscount(int price) {
        return price - discountAmount;
    }
}

// Null Object
class NoCoupon implements Coupon {
    @Override
    public int applyDiscount(int price) {
        return price; // 아무 할인도 하지 않음
    }
}

class OrderService {
    public int calculatePrice(int price, Coupon coupon) {
        return coupon.applyDiscount(price); // null 체크가 필요 없음
    }
}

public class Main {
    public static void main(String[] args) {
        OrderService orderService = new OrderService();

        Coupon coupon1 = new FixedCoupon(1000);
        System.out.println(orderService.calculatePrice(10000, coupon1)); // 9000

        Coupon coupon2 = new NoCoupon(); // null 대신 사용
        System.out.println(orderService.calculatePrice(10000, coupon2)); // 10000
    }
}
```

- **Null Object Pattern**은 `null`을 객체(Object)처럼 처리하여 예외 상황을 방지하고 코드의 간결성을 높이는 디자인 패턴이다.
- `null` 대신 아무런 동작도 하지 않는 **특별한 기본 객체**(`NoCoupon` 등)를 반환하도록 설계한다.
- 클라이언트 코드에서 번거로운 **`null` 체크 로직을 완전히 제거**할 수 있어, 불필요한 조건문을 줄이고 객체의 기본 동작을 정의하는 데 매우 유용하다.

## 5. Command Pattern

```java
interface Command {
    void execute();
}

class TurnOnCommand implements Command {
    private final Tv tv;

    public TurnOnCommand(Tv tv) {
        this.tv = tv;
    }

    @Override
    public void execute() {
        tv.turnOn();
    }
}

class TurnOffCommand implements Command {
    private final Tv tv;

    public TurnOffCommand(Tv tv) {
        this.tv = tv;
    }

    @Override
    public void execute() {
        tv.turnOff();
    }
}

class Tv {
    public void turnOn() {
        System.out.println("TV 전원 ON");
    }

    public void turnOff() {
        System.out.println("TV 전원 OFF");
    }
}

class RemoteController {
    private final Map<String, Command> commands = new HashMap<>();

    public void setCommand(String button, Command command) {
        commands.put(button, command);
    }

    public void press(String button) {
        Command command = commands.get(button);
        command.execute();
    }
}

public class Main {
    public static void main(String[] args) {
        Tv tv = new Tv();

        Command onCommand = new TurnOnCommand(tv);
        Command offCommand = new TurnOffCommand(tv);

        RemoteController remote = new RemoteController();
        remote.setCommand("ON", onCommand);
        remote.setCommand("OFF", offCommand);

        remote.press("ON");
        remote.press("OFF");
    }
}
```

- **커맨드 패턴(Command Pattern)** 은 요청이나 작업을 독립적인 객체로 캡슐화하여 처리하는 디자인 패턴이다.
- 작업을 호출하는 객체(Invoker)와 작업을 실제 수행하는 객체(Receiver)를 완전히 **분리**하는 것이 가장 큰 특징이다.
- 기존 코드를 거의 변경하지 않고 새로운 명령(`Command` 구현체)만 추가하면 되므로 **확장성**이 매우 뛰어나다.
- 각각의 기능이 클래스 단위로 명확하게 분리되어, 향후 특정 기능을 수정할 때 **수정해야 할 클래스가 아주 명확**해지는 장점이 있다.
- 하지만 간단한 작업에도 인터페이스와 여러 구현체, 호출 관리 클래스 등을 생성해야 하므로 전체적인 코드의 **복잡성이 증가**한다는 치명적인 단점도 존재한다.
- 따라서 단순한 `if`문으로 해결 가능한 로직인지, 향후 기능 확장과 명확한 분리가 필요한 상황인지 고려하는 **설계의 트레이드 오프** 판단이 중요하다.
