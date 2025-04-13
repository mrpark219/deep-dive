# 데이터 동기화를 위한 Kafka 활용

## 1. Apache Kafka

- Apache Software Foundation의 오픈 소스 메시지 브로커 프로젝트이다.
- 원래 링크드인(LinkedIn)에서 개발되었으며, 2011년에 오픈 소스화되었다.
- 2014년 Kafka 개발자들이 Confluent라는 회사를 창립하여 Kafka 개발에 집중하고 있다.
- Kafka는 실시간 데이터 스트리밍을 처리하기 위한 플랫폼으로, **높은 처리량과 낮은 지연 시간**을 제공한다.

### 1.1 기존 아키텍처의 문제점

- 서비스 간 직접 연결하는 End-to-End 방식은 **복잡성이 높고 확장에 불리**하다.
- 데이터 연동 방식이 제각각이기 때문에 새로운 시스템을 추가하거나 변경하기 어렵다.
- 데이터 흐름을 통제하고 관리하기 힘들며, 장애 전파 가능성도 높다.

### 1.2 Kafka를 활용한 아키텍처

- Kafka는 **모든 시스템이 Kafka를 중심으로 데이터를 주고받도록 구성된 중앙 메시지 허브 역할**을 수행한다.
- **Producer-Consumer 구조**로 송신자와 수신자를 분리해준다.
- 하나의 메시지를 여러 Consumer가 처리할 수 있어 확장이 용이하다.
- **고성능 처리**를 위해 내부적으로 메시지를 최적화된 구조로 저장한다.
- **Scale-out이 가능**하며, 다양한 생태계(Ecosystem) 도구와 연동된다.

## 2. Kafka Broker

- Kafka 브로커는 **Kafka 서버 애플리케이션이 실행 중인 인스턴스**를 의미한다.
- 일반적으로 **3개 이상의 브로커로 클러스터를 구성**한다.
- Kafka는 **Zookeeper**와 함께 동작하며, Zookeeper는 메타데이터를 관리한다.
  - Broker ID, Controller ID 등의 정보를 저장한다.

### 2.1 Controller

- 클러스터 내 브로커 중 하나는 Controller 역할을 맡는다.
- Controller의 역할:
  - **각 브로커에 파티션 할당**
  - **브로커의 상태를 모니터링하고 장애 복구를 수행**
  - 클러스터의 리더로서 파티션 리더 선출, 장애 브로커 재배치 등을 수행한다.

## 3. Kafka Client

- Kafka Client는 **Kafka 클러스터에 명령을 내리거나 데이터를 송수신하기 위한 라이브러리**이다.
- Kafka는 Producer, Consumer, Admin Client와 같은 다양한 **클라이언트 API**를 제공한다.
- Kafka Client는 **독립적인 실행 프로그램이 아닌 라이브러리 형태**이기 때문에, 사용하려면 **별도의 애플리케이션이나 프레임워크 위에 구현**하여 사용해야 한다.
- Kafka Client를 활용하면 다음과 같은 작업이 가능하다:
  - 메시지를 Kafka 토픽으로 전송 (Producer)
  - Kafka 토픽에서 메시지를 소비 (Consumer)
  - Kafka 클러스터나 토픽 관련 설정을 관리 (Admin Client)

## 4. Kafka Connect

- Kafka Connect는 **Kafka와 외부 시스템 간의 데이터를 Import/Export할 수 있는 도구**이다.
- **코드를 작성하지 않고 설정 파일(Configuration)만으로 데이터를 이동**시킬 수 있다.
- **Standalone Mode**와 **Distributed Mode**를 지원한다.
  - Standalone Mode: 단일 프로세스에서 실행.
  - Distributed Mode: 여러 워커 노드로 구성된 클러스터 환경에서 실행.
- **RESTful API**를 통해 커넥터를 설정하고 관리할 수 있다.
- **Stream 또는 Batch 형태로 데이터 전송**이 가능하다.
- 다양한 외부 시스템과 연동할 수 있도록 **커스텀 Connector를 지원**한다.
  - File, S3, MySQL, PostgreSQL 등.
- 데이터 흐름 방향에 따라 역할이 나뉜다:
  - **Kafka Connect Source**: 외부 시스템에서 Kafka로 데이터를 가져온다.
  - **Kafka Connect Sink**: Kafka에서 외부 시스템으로 데이터를 보낸다.
