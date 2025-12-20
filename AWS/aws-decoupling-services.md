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
