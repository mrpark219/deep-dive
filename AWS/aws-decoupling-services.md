# AWS 디커플링 서비스

## 1. 디커플링

- 아키텍처에서 구성 요소 간의 **의존성**을 낮추는 방법이다.
- 아키텍처의 **유연성**을 높이고 효율적으로 최적화할 수 있다.

### 1.1. 긴밀한 결합(Tight Coupling)

- 다른 주체와 단단하게 **얽힌 상태**이다.
- 구성 요소 간에 높은 **의존성**을 가지고 있어 변경이 쉽지 않다.

### 1.2. 느슨한 결합(Loose Coupling)

- 다른 요소와 얽히지 않고 **연결**되어 있는 상태이다.
- 구성 요소 간에 낮은 **의존성**을 가지고 있어 변경이 쉽고 **유연**하다.

### 1.3. AWS의 디커플링 서비스

- **Amazon SQS**이다.
- **Amazon SNS**이다.
- **Amazon Kinesis**이다.
- **Amazon MQ**이다.
- **Amazon MSK(Kafka)** 이다.
- 기타 다양한 **이벤트 기반 기능**들(S3 Event, EventBridge, DynamoDB Stream 등)이 있다.

## 2. Amazon SQS

- AWS에서 제공하는 **큐 서비스**이다.
  - **큐 서비스**는 아키텍처에서 주로 서비스 간 **데이터**를 전달할 때 활용한다.
  - 하나의 메시지를 **한 번만 처리**하는 것을 목표로 한다.
- 주요 사용 사례
  - 하나의 서비스에서 다른 서비스로 **데이터**를 전달하고 싶을 때 사용한다.
  - 아키텍처에서 AWS 서비스들의 **느슨한 연결(Loose Coupling)** 을 수립할 때 사용한다.
  - 메시지의 **처리 순서** 및 **처리 여부**를 보장하고 싶을 때 사용한다.
- **사용 방식**은 다음과 같다.
  - 메시지를 받아 **임시로 저장**(최대 14일)하고, 요청이 오면 들어온 순서대로 **전달**한다.
  - 전달받은 서비스가 메시지 처리 후 **완료 요청**을 보내면 큐에서 **삭제**한다.
  - 오류 등으로 처리를 못하면 메시지를 **반환**한다.
  - 메시지는 여러 주체가 요청하더라도 **한 번만 전달**되도록 처리한다.
- **Public 서비스**이다. 즉 AWS 외부에서 **인터넷**을 통해 사용 가능하다.
- **Serverless 서비스**로 분류된다.
  - **고가용성**이 자체적으로 확보되어 있다.
  - 별도의 **인프라 관리**가 불필요하다.
- **요청 횟수**, **메시지 크기** 등으로 과금된다.

### 2.1. SQS 구성 요소

- **메시지(Message)**: SQS에서 전달하는 **데이터의 단위**이다.
- **Producer**: 메시지를 생산하고 SQS에 **전달**하는 주체이다.
- **Queue**: 메시지를 **저장**하고 메시지를 **Consumer**에게 전달하는 다양한 기능을 담당한다.
- **Consumer**: 메시지를 받아 처리하고 **소비(삭제)** 하는 주체이다.
- **Dead Letter Queue**: 처리에 **실패**한 메시지를 모아둔 **2차 Queue**이다.
- **액세스 정책**: SQS에 접근할 수 있는 주체에 대한 **권한 설정 정책**이다.

### 2.2. SQS Queue의 타입

- **Standard Queue**
  - **순서**를 보장하지 않는다.
  - 메시지를 여러 번 전달할 가능성이 존재한다(**At Least Once Delivery**).
    - 여러 번 전달될 가능성이 있으므로 **멱등성(Idempotency)** 확보가 필요하다.
    - **멱등성**: 로직이 여러 번 수행되더라도 **동일한 결과**를 보장할 수 있는 성질이다.
- **FIFO Queue**
  - **순서**를 보장하며 메시지를 단 **한 번만** 전달하도록 보장한다(**Exactly Once Delivery**).
  - Standard에 비해 **처리량(성능)**의 제약이 있다.

### 2.3. SQS 메시지

- SQS에서 **전달**하는 데이터의 단위이다.
- **최소 크기**는 1바이트, **최대 크기**는 **1MiB(1,048,576바이트)** 이다.
  - 1MiB보다 큰 메시지는 **Amazon SQS 확장 클라이언트 라이브러리**를 사용하여 **S3**에 저장 후 참조하는 방식으로 전송 가능하다(최대 **2GB**).
- 최대 **14일**까지 저장 가능하다.

#### SQS 메시지의 상태

- **Stored**: Producer가 메시지를 SQS Queue에 전달을 완료하여 **대기 중**인 상태이다.
- **In Flight**: Consumer가 메시지를 가져와서 **처리 중**인 상태이다.
  - **Standard**와 **FIFO** 모두 최대 **120,000개**까지 가능하다.(과거 FIFO는 20,000개까지 가능했다.)
- **Deleted**: Consumer가 메시지 내용을 처리한 후 **삭제**한 상태이다.

#### SQS 메시지의 구성

- **Message Body**: 실제 메시지의 **내용(String)**이다.
  - 최대 **1MiB**이다(Message Attribute 포함).
- **Message Attribute**: **Key-Value** 형식의 메타데이터로 Body에 포함되지 않는 **추가적인 데이터**이다.
  - 주로 **분류**, **필터링**, Body를 처리하기 위한 **Context** 등에 활용한다.
  - 최대 **10개**까지 지정 가능하다.
- **Message System Attribute**: AWS 서비스에서 활용하기 위한 **Metadata**이다.
  - 현재는 AWS X-ray를 위한 AWSTraceHeader만 지원한다.
- **ReceiptHandle**: 메시지를 **삭제**하기 위한 **키**이다.
- **기타 정보**: **Region**, **Event Source**, **MD5**, **Timestamp**, **해시** 등이 포함된다.
- **FIFO 전용**: Queue 타입이 FIFO인 경우 **Message Group ID**, **Deduplication ID**가 포함된다.

### 2.4. Producer

- 메시지는 **API**를 사용하여 **Push** 가능하다.
  - 이때 **Queue URL**을 알고 있어야 정확한 SQS Queue를 판별하여 메시지를 전송할 수 있다.
- **Queue URL 형식**: `https://sqs.<region>.amazonaws.com/<account-id>/<queue-name>`
- **IAM 엔티티**로서 권한이 있거나 **리소스 기반 정책**으로 권한 확보가 필요하다(`sqs:SendMessage`).
- 다양한 AWS 서비스와 **이벤트**로 연동된다.
  - 기본적으로 연동 가능하거나 **EventBridge**를 통해 연동한다.

### 2.5. Consumer

- 권한이 있는 상태에서 주기적으로 **API**를 통해 Queue에 메시지를 요청(**Poll**)해서 처리한다.
  - 메시지를 받아오고 삭제하는 **권한**이 필요하다(`sqs:ReceiveMessage`, `sqs:DeleteMessage`).
- 기본적인 워크플로우(Workflow)는 **Poll -> 처리 -> 삭제** 순서이다.
- **Visibility Timeout** 내에 메시지를 받아 처리하고 완료 시 **삭제**해야 한다.
  - 삭제하지 않으면 Visibility Timeout 이후 다시 Queue로 **반환**된다.
  - 필요하다면 Visibility Timeout을 늘려 **추가 시간**을 확보 후 처리한다.
- **두 가지 Polling 방법**이 존재한다.
  - **Short Polling(기본)**: 메시지 요청 시 Queue의 **일부**를 검색하여 메시지를 찾으면 즉시 전달한다. 메시지를 못 찾아도 응답을 바로 전달한다.
  - **Long Polling**: 메시지 요청 시 **전체 Queue**를 검색하여 적어도 하나의 메시지를 찾아서 전달한다. 메시지가 없다면 찾을 때까지 기다리거나, **Timeout**을 넘기면 그때 응답을 전달한다(1~20초 대기 설정 가능).
    - **False Empty Response**를 방지하고, **요청 횟수**를 감소시킨다.
    - **Batch 처리**가 가능하다(한 번에 최대 10개 요청).

#### Visibility Timeout

- 메시지 요청 이후 다른 주체가 해당 메시지를 요청할 수 없는 **기간**이다.
  - 일종의 **Lock** 기능으로, 해당 기간 동안 다른 주체는 메시지 요청이 불가능하다.
- **기본 30초**이며, 최소 0초에서 최대 **12시간**까지 설정 가능하다.
- Visibility Timeout 기간 내에 처리(삭제)되지 않은 메시지는 자동으로 다른 주체에게 **공개(재처리 가능)** 된다.
  - 메시지를 처리하기 충분하되, 오류 상황에 적절히 대응할 수 있는 시간 설정이 필요하다.
- 기본적으로 **Queue 단위**로 설정하나 **메시지 단위**로도 변경 가능하다.
  - 즉 필요하다면 런타임에 Visibility Timeout **수정**이 가능하다.
- Visibility Timeout이 만료되면 메시지는 다시 **처리 가능한 상태**로 돌아간다.

### 2.6. SQS 보안

- **Access Policy**: 어떤 주체가 SQS Queue에 접근하여 메시지를 보내거나 가져올 수 있는지 정의하는 **리소스 기반 정책**이다.
  - 주체에게 IAM 권한이 없더라도 **Access Policy**에 권한이 명시되어 있으면 Queue 사용이 가능하다.
- **암호화**를 지원한다.
  - **SQS-SSE**(Encryption at rest) 및 **SQS-KMS**를 지원한다(S3와 유사).
  - **Message Body**만 암호화하며, Metadata, Timestamp 등은 암호화하지 않는다.

#### IAM 정책의 종류

- **Identity-based policies(자격 증명 기반 정책)**
  - **자격 증명**(IAM 유저, 그룹, 역할)에 부여하는 정책이다.
  - 해당 자격 증명이 **무엇을 할 수 있는지** 허용한다.
- **Resource-based policies(리소스 기반 정책)**
  - **리소스**(S3, SQS, VPC Endpoint, KMS 등)에 부여하는 정책이다.
  - 해당 리소스에 **누가 무엇을 할 수 있는지** 허용 가능하다.
  - 예: SQS 대기열에 **Lambda Service**가 접근 가능하도록 설정한다.

### 2.7. Dead Letter Queue (DLQ)

- 시스템 오류로 처리할 수 없는 메시지를 임시로 저장하는 특수한 유형의 **메시지 대기열**이다.
- 설정한 **재시도 횟수(MaxReceiveCount)** 보다 많이 실패했을 경우 DLQ로 전달된다.
- 추후 **재처리(Redrive)** 가 가능하다.

### 2.8. Monitoring

- 기본적으로 **CloudTrail**로 API 로깅이 가능하다.
- **CloudWatch** 기본 지표로 다양한 Metric을 제공한다.
- **주요 메트릭**은 다음과 같다.
  - **ApproximateAgeOfOldestMessage**: 가장 오래된 메시지의 **나이**(근사값)이다.
  - **ApproximateNumberOfMessagesNotVisible**: 현재 **In-Flight** 중(처리 중)인 메시지의 수(근사값)이다.
  - **ApproximateNumberOfMessagesVisible**: 현재 **Stored**, 즉 대기 중인 메시지의 수(근사값)이다.
  - **NumberOfEmptyReceives**: 메시지 요청 시 **빈 응답**이 전달된 횟수이다.
  - **NumberOfMessagesReceived**: 메시지 요청에 따라 **전달된** 메시지의 개수이다.
  - **NumberOfMessagesSent**: Queue에 **도착**한 메시지의 개수이다.
  - **SentMessageSize**: Queue에 도착한 메시지의 **크기**이다.

## 3. Amazon SNS(Simple Notification Service)

![https://docs.aws.amazon.com/ko_kr/sns/latest/dg/images/sns-delivery-protocols.png](./images/aws-decoupling-services/2025-12-21-16-14-07.png)

- 애플리케이션 간(**A2A**) 및 애플리케이션과 사용자 간(**A2P**) 통신 모두를 지원하는 **완전 관리형 메시징 서비스**이다.
- **Pub/Sub** 기반의 메시징 서비스이다.
  - 하나의 메시지를 여러 서비스에서 **처리**한다.
- **주요 사용 사례**
  - 하나의 메시지를 다수의 주체에서 처리하고 싶을 때(**Fan Out**) 사용한다.
  - 서비스에서 서비스로 메시지를 **전달**하고 싶을 때 사용한다.
  - **FIFO** 토픽을 활용하여 메시지를 **아카이빙**하거나 **리플레이(Replay)** 하여 동일한 로직을 다시 처리하고 싶을 때 사용한다.
- **사용 방식**
  - 하나의 **토픽(Topic)**을 여러 주체가 **구독(Subscribe)** 한다.
  - 토픽에 전달된 내용을 구독한 모든 주체가 전달받아 처리하므로, 하나의 메시지를 **여러 주체**가 처리하게 된다.
  - 다양한 **프로토콜** 및 서비스를 지원한다(**이메일**, **HTTP(S)**, **SQS**, **SMS**, **Lambda**).
  - **FIFO** 토픽의 경우 **메시지 리플레이** 기능을 제공한다.

### 3.1. Amazon SNS와 SQS의 차이

| 내용              | SNS                                                | SQS                                                     |
| :---------------- | :------------------------------------------------- | :------------------------------------------------------ |
| **목적**          | 여러 서비스에 메시지를 **전달**하기 위해 사용한다. | 특정 작업을 다음 서비스로 넘겨주기 위해 **버퍼링**한다. |
| **메시지 처리**   | 하나의 메시지를 **여러 서비스**에서 처리한다(1:N). | 하나의 메시지는 **한 번만** 처리한다(1:1).              |
| **메시지 보관**   | 기본적으로 **보관 불가능**하다(즉시 전달 시도).    | 최대 **14일** 보관 가능하다.                            |
| **전달 방식**     | **PUSH** 방식이다.                                 | **PULL**(Polling) 방식이다.                             |
| **아키텍처 활용** | **Fan Out**                                        | **디커플링(Decoupling)**                                |

### 3.2. Amazon SNS의 구성요소

- **토픽(Topic)**: SNS의 **커뮤니케이션 채널**이다.
- **구독(Subscription)**: 토픽으로 들어온 메시지를 받아볼 수 있는 **리소스**이다.
- **Publisher**: 메시지를 생산하고 SNS에 **전달**하는 주체이다.
- **Subscriber**: 실제 구독으로부터 메시지를 받아서 **처리**하는 주체이다.
- **메시지**: SNS에서 **전달**하는 데이터이다.
- **액세스 정책**: SNS에 접근할 수 있는 주체에 대한 **권한 설정 정책**이다.

### 3.3. Amazon SNS Topic / Subscription

- 토픽에 메시지를 발행(**Publish**)하면 토픽을 구독(**Subscribe**)한 모든 대상에게 메시지를 **발송**한다.
- 하나의 메시지를 다수의 대상이 처리(**Fan Out**)한다.
- **구독 프로토콜**은 다음과 같다.
  - **이메일**
  - **HTTP(S)**
  - **SQS**
  - **SMS**
  - **Lambda**
  - **Kinesis Data Firehose**
- 최초 구독 시 **확인(Confirmation)** 이 필요하다(확인 이메일, Lambda의 확인 이벤트 등).

### 3.4. Amazon SNS 메시지의 구성(Standard)

- **Message Body**: 실제 메시지의 **내용(String)** 이다.
  - **제목**과 **내용**으로 구성된다.
  - 최대 **256KB**이다(Message Attribute 포함).
  - 256KB보다 큰 콘텐츠의 경우 S3에 저장 후 버킷/키 정보만을 저장하는 방식으로 전달 가능하다.
  - **Raw Message Delivery**: SNS 메시지 포맷(JSON)을 따르지 않고, 전달받은 메시지 본문을 **그대로** 전달하는 기능이다.
    - 주로 S3 로깅, SQS 등 메시지를 파싱하지 않고 있는 그대로 처리해야 할 경우 활용한다.
- **Message Attribute**: **Key-Value** 형식의 메타데이터로 Body에 포함되지 않는 **추가적인 데이터**이다.
  - 주로 **분류**, **필터링**, Body를 처리하기 위한 **Context** 등에 활용한다.
    - 예: 영상 처리 알고리즘 명시, 분류를 위한 Tag 정의 등
  - 최대 **10개**까지 지정 가능하다.
- **TTL**: **모바일 푸시 알림** 전용 기능이다.
- **기타**: **Timestamp**, **토픽 ARN**, **시그니처**, 구독 해제를 위한 **URL** 등이 포함된다.

### 3.5. Amazon SNS 메시지 필터링

- 메시지 필터링을 통해 모든 메시지 대신 **특정 메시지**만 수신 가능하다.
- 각 **Subscription Filter Policy**를 설정하여 동작한다.
  - **Filter Policy**: 메시지의 Body 혹은 Attribute 단위로 원하는 메시지를 **매칭**한다.
  - 매칭 시 메시지를 **발송**하고, 불일치 시 메시지를 **전달하지 않는다**.
- **필터링 가능 대상**은 **Message Body**와 **Message Attributes**이다.
  - **AND**, **OR** 조건 사용이 가능하다.

### 3.6. Amazon SNS 보안

- **Access Policy**: 어떤 주체가 SNS에 접근하여 메시지를 보내거나 구독할 수 있는지 정의하는 **리소스 기반 정책**이다.
  - 주체에게 IAM 권한이 없더라도 **Access Policy**에 권한이 명시되어 있으면 **권한 부여**가 가능하다.
  - 예: IAM 사용자에 아무 권한이 없더라도 Access 정책에 권한이 명시되어 있으면 SNS로 **퍼블리시**가 가능하다.
- **암호화**를 지원한다.
  - **KMS**를 활용하여 **Server Side Encryption**을 구현한다.
  - **Body**만 암호화하며, **Metadata**, **Timestamp** 등은 암호화하지 않는다.

### 3.7. Amazon SNS Monitoring

- 기본적으로 **CloudTrail**로 API 로깅이 가능하다.
- **CloudWatch** 기본 지표로 다양한 Metric을 제공한다.
- **주요 메트릭**은 다음과 같다.
  - **NumberOfMessagesPublished**: **발행된** 메시지 숫자이다.
  - **NumberOfNotificationsDelivered**: 구독 대상에게 **전달된** 메시지 숫자이다.
  - **PublishSize**: 도착한 메시지의 **크기**이다.

### 3.8. Amazon SNS 기타 기능

- **Amazon SNS Data Protection**: SNS 메시지 중 **민감한 정보**를 감지하거나 검열해 주는 서비스이다.
  - **Data Protection Policy**를 기반으로 필터링한다.
  - **Inbound Message** / **Outbound Message** 모두 확인한다.
- 메시지 **순서 보장** 및 **중복 제거**를 지원한다(FIFO 토픽).
- 메시지 **보관** 및 **리플레이** 기능을 제공한다(FIFO 토픽).

### 3.9. Amazon SNS에서 SMS를 보낼 때 주의 사항

- **특정 리전**에서만 문자 메시지(SMS) 발송이 가능하다.
  - **서울 리전(ap-northeast-2)** 에서는 사용이 불가능하다(도쿄, 버지니아 등 사용 필요).
- Sandbox 모드 해제 및 지출 한도 상향 등을 위해 **서포트 케이스** 요청이 필요하다.
- 승인까지 **수 주**의 기간이 소요될 수 있으므로 **사전 준비**가 필수적이다.

## 4. FIFO

### 4.1. SQS FIFO

- 기본 SQS(Standard)와 달리 메시지의 **순서**를 보장하며, 단 **한 번**만 전달한다(**Exactly-Once Processing**).
- 대신 **성능(처리량)** 저하가 존재한다.
  - **High Throughput** 모드로 어느 정도 완화 가능하다.
- Queue 이름이 반드시 `.fifo`로 끝나야 한다.

#### Deduplication ID (중복 제거 ID)

- SQS에서 각 메시지의 **중복 여부**를 판단하기 위한 **고유 토큰**이다.
- 메시지를 전송할 때 부여하며, 특정 **Deduplication ID**를 가진 메시지가 SQS에 도달하면 **5분** 동안 같은 ID를 가진 메시지는 무시된다.
  - API 호출은 **성공**으로 반환되지만, 메시지는 큐에 **저장되지 않는다**.
- **두 가지 제공 방식**이 있다.
  - **Content-Based**: SQS가 자동으로 메시지 **Body**의 **SHA-256 해시**를 생성하여 ID로 사용한다.
    - **Message Attribute**는 해시 생성에 포함되지 않는다.
  - **Explicit**: **Producer**가 직접 Deduplication ID를 생성해서 전달한다.

#### Message Group ID

- **FIFO Queue** 내부의 일종의 **채널** 개념이다.
- **Message Group ID** 단위로 **순서 보장** 및 **전달**이 이루어진다.
  - 서로 다른 Message Group ID 간에는 **순서**가 보장되지 않는다.
- SQS FIFO에서는 동일한 Message Group ID를 가진 메시지는 동시에 **하나**만 처리 가능하다.
  - 하나의 Message Group에서 앞선 메시지가 처리(삭제)되지 않으면 뒤따르는 메시지들은 모두 **대기**한다.
- SNS FIFO에서 **SQS FIFO**로 전달 시 Message Group ID도 함께 **전달**된다.

### 4.2. SNS FIFO

- SNS의 메시지 전달을 **FIFO**로 처리할 수 있는 모드이다.
- **순서 보장**과 **중복 제거**라는 두 가지 효과가 있다.
- **SQS FIFO** 및 **Standard Queue**와만 연동 가능하다.
  - 이메일, 휴대폰(SMS), HTTP 엔드포인트 등 **일반적인 Subscriber**와는 연동이 불가능하다.
- **메시지 그룹**, **메시지 필터링** 등의 기타 기능을 제공한다.
- 주제(Topic) 이름이 반드시 `.fifo`로 끝나야 한다.

#### Message 순서

- 일반적인 **SNS + SQS** 조합은 순서를 **보장하지 않는다**.
  - 즉 메시지가 발송된 순서와 별도로 대상이 수신한다.
  - 순서를 보장하기 위해서는 **SNS FIFO + SQS FIFO** 조합이 필요하다.
- 각 Subscription에 전달되는 메시지별 순서는 다를 수 있지만, **받는 메시지의 순서**는 동일하다.
- **Message Sequence Number**: 연속적이지는 않지만 항상 **증가**하는 번호이다.
  - SNS FIFO에서 메시지를 받아 발송할 때 부여한다.
  - **Message Body**에 포함되나, **Raw Message Delivery** 활성화 시에는 포함되지 않는다.

#### Message Group ID

- SNS/SQS FIFO 내부의 일종의 **채널** 개념이다.
- **Message Group ID** 단위로 **순서 보장** 및 전달이 이루어진다.
- SNS FIFO에서 Message Group ID를 전달했을 때, 대상이 **SQS FIFO**라면 Message Group ID가 같이 전달된다.

#### Filtering

- **메시지 필터링**을 통해 모든 메시지 대신 **특정 메시지**만 수신 가능하다.
- 각 **Subscription Filter Policy** 설정이 가능하다.
  - **Filter Policy**: 메시지의 **Body** 혹은 **Attributes** 단위로 원하는 메시지를 매칭한다.
  - 매칭 시 메시지를 **발송**하며, 불일치 시 메시지를 **전달하지 않는다**.

#### Deduplication ID

- **Deduplication ID**를 기반으로 중복된 메시지를 제거한다.
  - **5분** 이내에 같은 Deduplication ID를 가진 메시지가 전달되면 요청은 성공하지만 실제로는 **전달되지 않는다**.
  - SNS가 전달된 **이후에도** ID는 트래킹된다.
- SNS에 전달된 메시지 **Body**를 기반으로 Deduplication ID 생성이 가능하다(**Content-Based**).
  - 메시지 Body의 **Hash**를 기반으로 ID를 생성한다.
  - **Attribute**는 Hash에 반영하지 않는다.

#### Message Archive/Replay

- SNS FIFO에 전달된 메시지를 **저장**하고 필요에 따라 **Replay**가 가능하다.
- 사용 사례
  - 메시지 **전달 과정**의 오류를 복구할 때 사용한다.
  - 기존 애플리케이션의 **장애 복구**에 사용한다.
  - 신규 애플리케이션의 **Sync**를 맞출 때 사용한다.
- **메시지 보관 기간**은 **1~365일** 설정 가능하다.
- 저장 시 및 처리 시 **추가 보관 비용**이 발생한다.
- **Replay** 시 **시간**을 정해서 Replay가 가능하다.
