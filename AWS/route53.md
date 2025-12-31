# Route 53

## 1. Amazon Route 53

- 높은 가용성과 확장성이 뛰어난 클라우드 **Domain Name System(DNS)** 웹 서비스이다.
- AWS의 **DNS 서비스**이다.
  - DNS에서 사용하는 **포트 숫자(53)** 에서 유래했다.
  - **도메인**을 **IP 주소** 및 AWS의 **리소스**로 연결해 주는 서비스이다.
- 기본적으로 **고가용성**을 갖춘 **글로벌 서비스**이다.
- 주요 기능
  - **도메인 관리**(등록, 레코드 연결 등)를 수행한다.
  - **Health Check 기능**: 주기적으로 지정된 주소에서 정상적인 응답을 받는지 **확인**한다.
  - 다양한 **라우팅 정책**을 제공한다.
  - **하이브리드 아키텍처** 환경에서 **내부 도메인**의 활용을 지원한다.
- **주요 비용**은 **호스팅 영역**, **DNS 쿼리**, 기타 기능(**Health Check** 등)에서 발생한다.

## 2. Amazon Route 53 주요 개념

- **도메인**: 대상의 IP 주소 등의 정보와 매핑되는 사람이 알아볼 수 있는 **문자열**이다.
  - **서브 도메인**: 도메인 중 스트링 앞에 추가 문자열이 붙은 **도메인**이다(예: `www.example.com`, `mail.example.com`).
  - **Apex 도메인(Zone Apex, Root Domain)**: 도메인 중 앞에 추가 문자열이 없는 순수한 **최상위 도메인**이다(예: `example.com`).
- **레코드(DNS Record)**: 도메인이 어떤 방식으로 트래픽을 대상에게 전달하는지 정의하는 **데이터**이다.
  - 레코드 종류, 대상 IP 주소 등의 정보를 **포함**한다.
  - **레코드 별 TTL**: 얼마나 오랫동안 다른 DNS 서버들이 이 레코드를 **캐싱**할지 결정한다.
    - TTL 값이 높다면 쿼리는 줄어들지만 업데이트 **배포 시간**이 증가한다.
- **Hosted Zone**: 레코드의 집합으로, 특정 도메인과 서브 도메인의 레코드를 모은 **컨테이너**이다.
  - Apex 도메인과 같은 이름을 **부여**한다.

### 2.1. 레코드(Record)의 종류

- **A(Address) Record**: 도메인을 **IPv4 주소**와 연결한다(`example.com` -> `192.33.22.11`).
- **AAAA(IPv6 Address) Record**: 도메인을 **IPv6 주소**와 연결한다.
- **CNAME(Canonical Name) Record**: 도메인을 **다른 도메인**과 연결한다.
  - 예: `www.example.com` -> `example.com`
  - 규칙상 **Apex 도메인**은 CNAME 사용이 **불가능**하다.
- **Alias(별칭) Record**: AWS Route 53에서만 지원하는 레코드 타입으로 도메인과 **AWS 리소스**를 연결한다.
  - 도메인을 **S3**, **CloudFront**, **ALB** 등과 연결한다.
  - **HTTPS Record**: HTTPS를 지원하는 리소스를 위해서 더 많은 정보를 제공해서 더 효율적인 **연결**을 지원한다.
- **NS(Name Server) Record**: 도메인의 **Authoritative DNS 서버**를 지정한다.
- **MX(Mail Exchange) Record**: 도메인과 **메일 서버**를 연결한다.
- **TXT(Text) Record**: 도메인에 관련된 **텍스트 기반 정보**를 연결한다.

### 2.2. Amazon Route 53 사용 과정

- **도메인 등록**을 수행한다(Route 53 혹은 다른 Domain Registrar).
  - `.kr` 등의 도메인은 Route 53에서 **등록 불가능**하다.
- **Hosted Zone 생성**을 수행한다.
  - Route 53에서 도메인을 등록하면 **자동**으로 Hosted Zone이 생성된다.
  - 다른 Domain Registrar에서 등록했다면 **수동**으로 Hosted Zone 생성 후 **DNS 연동**이 필요하다.
- **레코드 생성**을 수행한다.
  - AWS 리소스를 연결하려면 **Alias Record**를 사용한다.
  - 기타 필요에 따라 적절한 **레코드 타입**을 생성한다.
  - DNS 캐시 등의 이유로 전파까지 최대 **하루** 이상 소요될 수 있다.

### 2.3. Alias Record를 우선적으로 사용하는 이유

- **Apex 도메인** 연결이 가능하다.
  - `example.kr`로 ALB 혹은 S3 Static Hosting을 원할 경우 **CNAME**은 불가능하지만 **Alias**는 가능하다.
- **무료**이다.
  - Route 53은 쿼리(정보 조회)당 비용이 발생하나 **Alias Record**에 대한 쿼리는 무료이다.
- **HTTPS Alias Record**는 특히 더 **효율적인 연결**을 지원한다.
  - 기존 A/AAAA 레코드의 경우 클라이언트는 첫 요청에서 서버가 지원하는 **프로토콜**을 확인할 수 없다(여러 통신 이후 확인 가능).
  - **HTTPS Record**의 경우 첫 통신에서 지원 **프로토콜**까지 확인 가능하다.
    - 즉, 바로 지원하는 프로토콜로 **연결 수립**이 가능하다.

## 3. Amazon Route 53 Health Check

- Route 53에서 설정한 리소스(웹 애플리케이션, 서버 등)의 상태를 **모니터링**하는 기능이다.
  - **Routing Policy**에서 활용하거나 **Amazon CloudWatch 경보**를 설정하여 알림 처리가 가능하다.
- **Latency 정보** 확인이 가능하다.
  - **TCP Connection** 연결 수립까지 걸린 시간이다.
  - **HTTP/HTTPS First Byte**를 받기까지 걸린 시간이다.
    - **SSL/TLS Handshake**까지 걸린 시간이다.
- **CloudWatch Metric**으로 Health Check 상태 기록 및 알람이 가능하다.
  - **us-east-1** 리전 전용이다.

### 3.1. Health Check 모니터링 대상

- **특정 리소스**
- **다른 Health Check**
- **Amazon CloudWatch 경보**
- **Route 53 Application Recovery Controller**

#### 리소스(Resource)

- **IP 주소**, **도메인**에 주기적으로 요청을 보내 응답 여부를 확인하여 **가용성**을 확인한다.
- **상태 확인 방법**은 다음과 같다.
  - 전 세계 다수의 **Health Checker**에서 정해진 프로토콜 및 주기로 요청을 보내서 상태를 확인한다.
  - 주기는 **10초**(추가 요금) 또는 **30초**이며, Checker 별 Sync는 없다.
- **두 가지 지표**를 기준으로 상태를 판단한다.
  - **응답 속도**: HTTP(연결 수립) 4초, TCP 10초, HTTP String Match는 2초 안에 값을 확인해야 한다.
  - **응답 내용** 및 지정한 **실패 횟수**를 연속으로 넘었는지 여부이다.
- 해당 기준으로 전체 Health Checker의 **18% 초과**가 Healthy 상태를 유지해야 **Health 상태**로 판단한다.
- 주의 사항으로 AWS 외부의 엔드포인트를 대상으로 할 경우 **추가 요금**이 발생한다.

#### 리소스 지정 가능 값

- **모드**
  - **IP 주소**: **IPv4**, **IPv6**를 지원한다(로컬, Private, Multicast 등은 체크 불가능).
    - **HTTP/HTTPS**의 경우 Status 2xx, 3xx를 정상으로 판단한다.
    - **TCP** 연결 성공 여부를 확인한다.
  - **도메인**
    - **HTTP/HTTPS**의 경우 Status 2xx, 3xx를 정상으로 판단한다.
    - **IPv4**만 지원하며, **A Record** 외에는 Fail 처리된다.
- **매칭 문자열**: 응답의 **Body**에 특정 문자열이 있는지 확인한다.
- **Health Check Region**: 최소 **3개** 이상이어야 한다.
- 기타 **포트**, **경로**, **주기**, **SNI 지원**, **Latency Graphs**, **Inverse Check** 등을 설정할 수 있다.

#### 다른 Health Check (Calculated Health Check)

- **다른 Health Check**을 모니터링한다.
  - 예: 3개 이상의 설정한 **Health Check** 상태 검사가 실패할 경우 경보 또는 **Failover**를 수행한다.
- 상태를 정하기 위한 **Health Check 숫자** 지정이 가능하다.
  - **모든** Health Check가 성공이면 성공이다(AND).
  - 단 **하나**의 Health Check가 성공이면 성공이다(OR).
  - 지정한 Health Check 숫자 중 **N개 이상**이 성공이면 성공이다.

#### CloudWatch 경보

- **CloudWatch 경보 상태**를 모니터링한다.
- **주의 사항**은 다음과 같다.
  - CloudWatch 경보의 상태가 아닌 **직접 데이터**를 모니터링한다.
    - **CloudWatch 경보**보다 조금 더 민감하게 반응한다.
  - **Standard Resolution**(60초마다 수집) 경보만 모니터링 가능하다.
  - **Average**, **Minimum**, **Maximum**, **Sum**, **SampleCount**만 모니터링 가능하다.
  - **Math Metric**은 사용 불가능하다.
  - CloudWatch 경보가 변경되었을 경우 Route 53 Health Check를 **수동**으로 업데이트해야 한다.
- 데이터가 충분하지 않을 경우(**Insufficient Data** 상태) 상태 지정이 가능하다.
  - 예: Insufficient Data일 때 **Healthy**, **Unhealthy**, **Last Known Status**로 설정 가능하다.
