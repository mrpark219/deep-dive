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

## 4. Amazon Route 53 Routing Policy

- 도메인 레코드에 대상을 연결하는 방식을 다양한 방법으로 **지원**한다.
- 종류
  - **Simple** (단순)
  - **Failover** (장애 조치)
  - **Geolocation** (지리적 위치)
  - **Geoproximity** (지리적 근접)
  - **Latency** (지연 시간)
  - **IP-based** (IP 기반)
  - **Multivalue Answer** (다중 값 응답)
  - **Weighted** (가중치 기반)

### 4.1. Simple Routing Policy

- 가장 간단한 방식으로 하나의 레코드를 하나의 대상으로 라우팅하는 **방식**이다.
- **단일 서버**, **단일 리소스** 등의 라우팅을 위해 사용한다.

### 4.2. Failover Routing Policy

- 평소에는 **기본 대상(Primary)** 으로 라우팅하고, 기본 대상에 문제가 있을 때 **보조 대상(Secondary)** 으로 라우팅하는 정책이다.
- **Health Check**을 활용하여 상태를 확인한다.
  - 기본 대상의 Health Check이 **Fail 상태**일 경우 보조 대상으로 라우팅한다.
- **주요 사용 사례**는 다음과 같다.
  - **Active-Passive Failover**
    - 기본 대상이 대부분의 요청을 처리하고, 기본 대상의 실패를 대비해 보조 대상이 **준비**만 하는 Failover 방식이다.
    - 즉 평소에 대부분의 리소스를 기본 대상에 집중하고, 보조 대상은 **최소한의 리소스**로 장애를 준비한다.

### 4.3. Geolocation Routing Policy

- DNS Query가 발생한 **위치**에 따라 다른 응답을 보낼 수 있는 Routing Policy이다.
  - 즉 요청한 지점이 **지리적**으로 어디에 위치해 있는지에 따라 응답을 다르게 설정 가능하다.
  - 예: 동남아시아 지역에서 발생한 요청은 한국 리전, 유럽 지역에서 발생한 요청은 프랑크푸르트 리전으로 연결한다.
- **대륙**, **국가** 단위 설정이 가능하며, 미국의 경우 **주(State)** 기준으로 설정 가능하다.
  - 겹치는 범위라면 **더 작은 지역**을 우선 적용한다.
- 내가 지정한 지역의 요청은 내가 지정한 지역으로 **라우팅**한다.
- **주요 사용 사례**는 다음과 같다.
  - **언어별** 라우팅
  - **지역별 콘텐츠** 제공 구분
  - 예상 가능한 **부하**(인구 등)를 기반으로 인프라를 구축하고 유지할 때 사용한다

### 4.4. Geoproximity Routing Policy

- DNS Query가 발생한 위치와 **리소스의 위치**에 따라 다른 응답을 보낼 수 있는 Routing Policy이다.
  - 요청이 발생한 지점에서 가장 **가까운 위치**의 리소스로 라우팅한다.
  - 즉 요청한 지역과 리소스의 **거리 기반**이다.
- 추가적으로 **Bias(편향)** 를 지정해 지역 범위를 조절할 수 있다.
  - **Bias**: 특정 지역이 더 많은 범위 혹은 더 적은 범위를 커버하도록 조절하는 **보정값**이다.
- **주요 사용 사례**는 다음과 같다.
  - **지역별 콘텐츠** 제공 구분
  - **최소 지연 속도**로 라우팅

### 4.5. Latency-Based Routing Policy

- 유저 기준으로 가장 빠른 **Latency(네트워크 지연 시간)** 를 가진 레코드로 라우팅하는 정책이다.
  - 예: 도쿄 리전과 버지니아 리전에 ALB가 있을 때 서울 리전에서 요청한 경우, 서울->도쿄와 서울->버지니아를 비교하여 가장 빠른 리전으로 라우팅한다.
- **주의 사항**으로 AWS 데이터센터 간의 Latency를 기준으로 하기 때문에, AWS **외부의 소스**일 경우 정확도가 매우 떨어질 수 있다.
- **주요 사용 사례**는 다음과 같다.
  - 여러 리전 간에 **최적화된 유저 경험**을 위한 라우팅
  - **Active-Active Failover**
    - 평소에 **모든 리전이 트래픽을 처리**하다가, 특정 리전에 장애가 발생하면 자동으로 **차순위 Latency를 가진 리전으로 트래픽을 우회**시킨다.

### 4.6. IP-based Routing Policy

- **IP 기반**으로 라우팅을 조절하는 정책이다.
  - 기존의 Geolocation/Latency-Based 라우팅 등에 추가로 **네트워크 이해**를 바탕으로 정교한 라우팅 정책 구성이 가능하다.
- **CIDR Block Range** 별로 다른 라우팅 구성이 가능하다.
- **주요 사용 사례**는 다음과 같다.
  - 특정 네트워크를 **구분**하여 라우팅할 때 사용한다.
    - 예: 특정 **ISP 대역**만 분리하여 라우팅하고 싶은 경우
    - 예: **회사 IP**만 Dev 리전으로 라우팅

### 4.7. Multivalue Answer Routing Policy

- 한 번의 요청으로 **다양한 값**을 전달하는 정책이다.
  - **Health Check**와 연동하여 현재 원활한 상태인 값만 **선별적**으로 보내기가 가능하다.
- 주요 사용 사례
  - **로드 밸런싱**
  - 간단한 **Failover**

### 4.8. Weighted Routing Policy

- 다수의 리소스를 하나의 도메인으로 묶어 각 리소스에 **비중**을 두고 분배하는 정책이다.
  - 같은 이름과 타입의 레코드를 만들어 **비중(Weight)** 을 부여한다.
- **비중(Weight)** 은 **0~255**의 값으로, 높은 비중을 가진 레코드일수록 더 많은 트래픽을 분배한다.
- **주요 사용 사례**는 다음과 같다.
  - **로드 밸런싱**
  - **A/B 테스트**
  - **카나리 릴리즈(Canary Release)**
  - **Active-Active Failover**
