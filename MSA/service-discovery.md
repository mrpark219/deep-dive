# Service Discovery

## 1. Spring Cloud Netflix Eureka (Service Discovery)

- 마이크로서비스 간 통신을 위한 **서비스 등록 및 검색 기능을 제공하는 시스템**이다.
- Eureka는 **전화번호부**처럼 작동하여 서비스 이름(key)과 실제 위치(value)를 관리한다.
- 각각의 마이크로서비스는 **Eureka 서버에 자신을 등록**하며, 이를 통해 다른 서비스들이 해당 서비스의 위치를 검색할 수 있다.
- 내부 동작:
  1. 클라이언트가 API Gateway나 Load Balancer를 통해 요청을 보낸다.
  2. 해당 요청은 **Service Discovery**를 통해 전달 가능한 서버의 위치 정보를 확인한다.
  3. Load Balancer는 해당 정보를 바탕으로 **적절한 인스턴스에 요청을 전달**한다.
- 이 구조를 통해 **서비스 간의 위치 정보 변경이나 확장/축소에도 유연하게 대응**할 수 있다.
