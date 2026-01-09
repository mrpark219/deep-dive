# CloudFront

## 1. CloudFront

- 개발자 친화적 환경에서 짧은 지연 시간과 빠른 전송 속도로 데이터, 동영상, 애플리케이션 및 API를 전 세계 고객에게 안전하게 전송하는 **고속 콘텐츠 전송 네트워크(CDN) 서비스**이다.
- AWS에서 제공하는 **Content Delivery Network(CDN)** 이다.
  - **웹 페이지**, **이미지**, **동영상** 등의 콘텐츠를 본래 서버에서 받아와 **캐싱**한다.
  - 해당 콘텐츠에 대한 요청이 들어오면 캐싱해둔 콘텐츠를 **제공**한다.
  - 콘텐츠를 제공하는 서버와 실제 요청 지점 간의 지리적 거리가 매우 먼 경우, 요청 지점 근처의 **CDN**을 통해 빠르게 콘텐츠 제공이 가능하다.
- **엣지 로케이션**에서 데이터를 캐싱한다.
  - 따라서 원본이 변해도 캐싱이 만료되지 않는다면 유저가 보는 내용은 **변하지 않는다**.
- **글로벌 서비스**이며, 리전은 **us-east-1**으로 취급된다.
- **정적 콘텐츠**, **동적 콘텐츠** 모두 호스팅 가능하다.

## 2. CloudFront 용어 정리

### 2.1. 엣지 로케이션(Edge Location)

- AWS의 CloudFront 등 여러 서비스들을 가장 빠른 속도로 제공(캐싱)하기 위한 **거점**이다.
- **Global Accelerator**와 유저를 연결하는 거점이다.
- 전 세계 여러 장소에 흩어져 있다.
- **POP(Points Of Presence)** 라고도 부른다.

### 2.2. 정적(Static) 콘텐츠, 동적(Dynamic) 콘텐츠

#### 정적 콘텐츠

- 서버에 저장된 파일이 모든 사용자에게 **동일**하게 전달되는 콘텐츠이다.
- 매번 서버에 요청할 필요 없이 **캐싱** 가능하다.
- HTML, Javascript, 이미지, 글, 뉴스 등으로 구성된다.

#### 동적 콘텐츠

- 시간, 사용자, 입력 등에 따라 내용이 **변경**되는 콘텐츠이다.
- 매번 서버에 요청하여 내용을 구성하고 전달받아야 한다.
- PHP, ASP 등으로 서버에서 처리한다.
- 로그인이 필요한 내용, 게시판, 댓글 등이 해당한다.

### 2.3. 원본(Origin)

- 실제 콘텐츠가 존재하는 **서버**(S3, EC2 등)이다.

### 2.4. 배포(Distribution)

- CloudFront의 **CDN 구분 단위**로, 여러 엣지 로케이션으로 구성된 콘텐츠 제공 채널이다.

### 2.5. 동작(Behavior)

- 프로토콜, **캐싱 정책**, 로그 등 어떻게 콘텐츠를 전달할지 **설정**하는 기능이다.

### 2.6. Regional Edge Cache

- **Edge Location**의 상위 단위로, 좀 더 큰 **캐싱 거점**이다.

## 3. Origin

- 뷰어에게 보여줄 **콘텐츠의 원본**이 있는 거점이다.
- 크게 두 가지 종류(**Amazon S3**, **Custom Origin**)가 있다.
  - 기본적으로 배포(Distribution) 당 최대 **25개**까지 설정 가능하다.

### 3.1. S3 Origin

- **Amazon S3**를 Origin으로 설정해 콘텐츠를 제공하는 경우이다.
- 도메인 형식은 `{bucketname}.s3.{region}.amazonaws.com`이다.
  - 이 형식이 아닐 경우 **Custom Origin**으로 취급된다.
  - **S3 Static Hosting** 엔드포인트(`http://...`)를 사용하는 경우 **Custom Origin**으로 설정해야 한다.
- **S3 Origin**만의 추가 기능은 다음과 같다.
  - **OAC(Origin Access Control)** 또는 **OAI(Origin Access Identity)**를 통한 S3 접근 제한이 가능하다.
  - **POST/PUT** 등을 통해 직접 S3에 콘텐츠 업데이트가 가능하다.
  - **S3 Object Lambda** 등을 활용할 수 있다.

### 3.2. Custom Origin

- **S3 버킷**을 제외한 모든 Origin을 의미한다.
  - **MediaStore**, **S3 Static Hosting**, **Lambda Function URL**, **ALB**, **EC2** 및 기타 HTTP 소스가 해당된다.
- **HTTP** 혹은 **HTTPS**로 접근할지 선택 가능하다.
- **IP 주소**는 사용 불가능하며 **도메인**만 지정 가능하다.

### 3.3. Origin Group

- **Failover**를 대비하여 **Primary**, **Secondary** 두 Origin을 그룹으로 묶어 관리하는 기능이다.
- **Primary**에서 실패한 경우 자동으로 **Secondary**에 요청한다.
  - **실패 조건**은 다음과 같다.
    - 실패를 나타내는 **HTTP 코드**(500 등)가 반환된 경우
    - Primary와 **통신**을 할 수 없는 경우(타임아웃 등): 기본 **10초**(3번 시도)이며 조절 가능하다.
    - 요청의 **응답**이 늦어지는 경우: 기본 **30초**에서 최대 **60초**까지 조절 가능하다.
- **GET**, **HEAD**, **OPTION** 메서드에만 적용된다.
- Primary, Secondary 모두 실패한 경우 **커스텀 에러 페이지** 생성이 가능하다.

### 3.4. Origin Custom Header

- CloudFront에서 Origin에 요청 시 **커스텀 헤더**를 추가하여 전달 가능하다.
- 이미 요청에 해당 헤더가 포함되어 있으면 **덮어씌운다(Override)**.
- 최대 **10개**까지 설정 가능하며 증가 요청이 가능하다.
- 별도로 **추가 불가능한 헤더**가 존재한다([AWS 공식 문서 참조](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/add-origin-custom-headers.html)).

## 4. Caching

### 4.1. Caching 티어

- CloudFront의 캐싱은 **2티어** 구조이다.
- **요청 순서**: **Edge Location** -> **Regional Edge Cache** -> **Origin** 순이다.

### 4.2. Cache Key

- 요청에 따라 어떤 캐시 내용을 보여줄지를 결정하는 **정보의 조합**이다.
  - 각 오브젝트는 고유의 **Cache Key** 단위로 캐시된다.
- **Cache Hit**: 뷰어가 특정 Cache Key로 오브젝트를 요청하였을 때, Edge Location에서 해당 캐시 오브젝트를 가지고 있어 **원본 요청 과정 없이** 제공할 수 있는 상황이다.
  - Origin의 **부하**를 경감할 수 있다.
  - 더 **빠르게** 콘텐츠를 제공할 수 있다.
  - 즉 CDN이 **Cache Hit**이 많을수록 더 좋은 **퍼포먼스**를 제공한다.
- **주요 Cache Key 구성 요소**: **경로(Path)**(기본), **Query String**, **HTTP Header**, **Cookie** 등이 있다.

#### HTTP Header based Cache

- Cache Key 중 **HTTP Header**를 활용하는 방식이다.
- 활용 사례
  - **언어별** 캐싱
  - **Device Type별** 캐싱
    - CloudFront에서 별도로 **User-Agent** 기반으로 전용 헤더를 생성한다.
    - `CloudFront-Is-Desktop-Viewer`, `CloudFront-Is-Mobile-Viewer`, `CloudFront-Is-SmartTV-Viewer` 등
  - **지역별** 캐싱
    - CloudFront에서 전용 헤더 `CloudFront-Viewer-Country`를 생성한다.
- 헤더명은 대소문자를 구분하지 않지만 **값**은 구분한다.

#### Cookie Based Cache

- **Cookie**를 기반으로 콘텐츠 내용을 캐시한다.
  - Cookie를 활용하지 않는 HTTP 서버 혹은 S3에서 사용할 경우 **퍼포먼스만 저하**될 수 있다(Cache Hit율 하락).

### 4.3. Cache 만료

- 캐시된 Object는 일정 기간(**Time To Live: TTL**) 이후 만료된다.
  - 만료 후 요청이 올 경우 CloudFront는 Origin에 Object **갱신 여부**를 확인한다.
  - Origin이 **304 Not Modified**를 줄 경우 갱신이 필요 없다(캐시 유지).
  - **200 OK**와 파일을 줄 경우 갱신한다.

#### Cache TTL

- Cache Object를 얼마나 오래 보관할지에 관한 **설정**이다.
  - 기본 **24시간**이다.
  - 모든 CloudFront의 Object에 적용된다.
  - 파일 단위에서는 Origin에서 `Cache-Control` 헤더 혹은 `Expires` 헤더를 포함해서 **조절** 가능하다.
- **TTL의 종류**는 다음과 같다.
  - **Minimum TTL**: 파일 단위 컨트롤에서 줄 수 있는 **최소** TTL이다.
  - **Maximum TTL**: 파일 단위 컨트롤에서 줄 수 있는 **최대** TTL이다.
  - **Default TTL**: 별도의 설정이 없을 경우 부여되는 **기본** TTL이다.

#### Cache TTL 컨트롤

- 파일 단위에서는 Origin에서 `Cache-Control` 헤더 혹은 `Expires` 헤더를 포함해서 **조절** 가능하다.
- **Cache-Control**: 얼마나 오래 Object를 캐시하는지 기간을 설정한다.
  - **max-age**: CloudFront와 **브라우저** 둘 다 영향을 받는다.
  - **s-maxage**: **CloudFront**만 영향을 받는다.
  - **no-cache**, **no-store**: 캐싱하지 않는다(단 Minimum TTL이 0 이상일 경우 Min TTL로 최저 설정된다).
- **Expires**: Cache가 만료되는 **정확한 시각**을 설정한다.
  - CloudFront와 브라우저 모두 영향을 받는다.
- CloudFront의 **Min/Max TTL 범위** 안에서만 설정 가능하다.

### 4.4. Cache Policy

- 캐싱과 관련된 내용을 **정책**으로 정의하여 CloudFront에 적용 가능하다.
- 주요 설정
  - 어떤 **키**(HTTP Header, 쿠키, Query String 등)로 콘텐츠를 캐시하는지 설정한다.
  - 얼마나 오래 캐시하는지(**TTL**) 설정한다.
  - 콘텐츠 **압축 저장** 관련 설정을 한다.
- **두 가지 종류**가 있다.
  - **Managed**: AWS에서 직접 생성한 Policy로 다양한 상황을 위해 **미리 준비된** Policy이다.
  - **Custom**: 사용자가 **직접 설정**하는 Policy이다.

## 5. Behavior

- CloudFront의 요청이 어떻게 처리되는지 정의하는 다양한 **설정의 모음**이다.
- **경로 패턴(Path Pattern)** 단위로 Behavior를 구성한다.
  - 예: `files/*`, `files/*.png`, `*.png`
  - 경로 패턴 목록 중 **처음**으로 매칭된 패턴의 동작을 적용한다(우선순위 적용).
  - 신규 생성 시 `*`(Default)로 고정된다.
- 주요 구성 내용
  - **Origin**
  - **뷰어 설정(Viewer Settings)**
  - **Cache Policy**, **Response Headers Policy**, **Lambda@Edge** 연결

### 5.1. Viewer 설정

- **Viewer Protocol Policy**
  - CloudFront에 접근하는 **프로토콜**을 설정한다.
  - **HTTP and HTTPS**: 둘 다 허용한다.
  - **Redirect HTTP to HTTPS**: HTTP 요청을 HTTPS로 리다이렉트한다.
  - **HTTPS Only**: HTTPS 요청만 허용한다.
- **Allowed HTTP Methods**
  - **GET**, **HEAD**, **OPTIONS**, **PUT**, **POST**, **PATCH**, **DELETE**를 지원한다.
- **뷰어 액세스 제한(Restrict Viewer Access)**
  - **Signed URL(Presigned URL)** 또는 **Signed Cookie**로만 접근 가능하도록 할지 여부를 결정한다.

### 5.2. Policy 설정

- **Cache Policy**
  - 어떤 **키**(HTTP Header, 쿠키, Query String 등)로 콘텐츠를 **캐시하는지** 결정한다.
  - 얼마나 오래 캐시하는지(**TTL**) 설정한다.
  - 콘텐츠 **압축 저장(Gzip, Brotli)** 관련 설정을 포함한다.
- **Origin Request Policy**
  - Origin에 콘텐츠를 요청할 때 어떤 내용을 **전달**할 것인지(HTTP Header, Query String, Cookie) 설정한다.
  - **Cache Key**로 사용할지 여부와 **독립적**으로 설정 가능하다(즉, 캐시 키에는 포함하지 않으면서 Origin에는 데이터 전달 가능).
- **Response Headers Policy**
  - 클라이언트로 응답하는 과정에서 어떤 **HTTP Header**를 **제거**하거나 **추가**할지 설정한다.
  - 커스텀 헤더는 최대 **10개**까지 설정 가능하다.
  - **Authorization Header**는 별도로 처리가 불가능하다.

### 5.3. 기타 설정

- **Field Level Encryption**: CloudFront에서 Origin으로 데이터를 전송할 때 특정 필드를 **암호화**하여 콘텐츠를 보호하는 방법이다.
- **Lambda@Edge** 및 **CloudFront Functions** 연결이 가능하다.

## 6. Amazon CloudFront의 파일 관리

- CloudFront에서 파일은 크게 **두 가지** 방식으로 관리된다.
  - **싱글 파일(Single File)**: 파일명을 유지한 채로 캐시 만료 처리 및 업데이트를 수행한다.
    - 별도의 **클라이언트 업데이트**가 필요 없다.
    - 캐시 만료 전 제공 파일을 업데이트하려면 **Invalidation(무효화)** 이 필요하다.
  - **버저닝(Versioning)**: 파일 이름에 다양한 방법(Query String, 파일명 변경 등)으로 버전을 두어 관리한다.
    - 별도의 **Invalidation**이 필요 없다.
    - 파일 업데이트 시 **클라이언트 업데이트**가 필요하다.

### 6.1. Invalidation (무효화)

- 캐시 만료 전 파일을 강제로 **갱신**하는 기능이다.
- **버저닝**을 사용하지 않는 경우, 캐시 만료 전 새 파일을 배포하려면 Invalidation이 **필수적**이다.
- **경로(Path)** 기반으로 수행한다.
  - 예: `/images/image1.jpg`
  - 예: `/images/image*`
  - 예: `/images/*`
- 한 번에 최대 **3,000개**의 파일 경로까지 Invalidation 가능하다.
  - 예: 100개씩 30번 요청하거나 1,000개씩 3번 요청 가능하다.
- 한 달에 **1,000개**의 경로 Invalidation은 **무료**이다(계정 전체 **Distribution** 통합).
  - 무료 횟수 초과 시 경로당 **$0.005**가 부과된다.

## 7. Amazon CloudFront의 콘텐츠 보호

### 7.1. 콘텐츠 접근 제한

- CloudFront에서 콘텐츠에 대한 **접근**을 제한하는 기능이다.
  - **뷰어 접근 제한**: CloudFront에 접근하는 주체별로 다르게 콘텐츠의 접근 제한이 필요한 경우에 사용한다.
    - 예: 프리미엄 티어용 영상, 유저 전용 다운로드 이미지 등
  - **Origin 접근 제한**: CloudFront를 거치지 않고 직접 Origin에 접근하는 것을 막고 싶은 경우에 사용한다.

### 7.2. CloudFront에서 뷰어 접근 제한 (Signed URL/Cookie)

- 다음 **두 가지 방법**을 사용하여 제한한다.
  - **Signed URL**: 권한 정보가 담긴 **임시 URL**을 발급하여 뷰어에게 전달하고 콘텐츠를 다운로드할 수 있도록 허용한다.
    - URL당 **하나의 파일**만 사용 가능하다.
  - **Signed Cookie**: 뷰어가 권한을 행사해 다운로드할 수 있도록 콘텐츠 접근 권한을 가진 **Cookie**를 발급해 뷰어에 전달한다.
    - **다수의 파일**에 사용 가능하다.
  - 두 방식 모두 **만료 기간** 설정이 가능하다.

#### Signer

- Signed URL/Cookie를 만들 **권한**을 가진 주체이다.
- 두 가지 종류
  - **Trusted Key Group(추천)**: CloudFront에 Public/Private Key Pair 중 **Public Key**를 등록하고, 가지고 있는 **Private Key**로 Presigned URL/Cookie를 생성하는 방식이다.
  - **AWS Account(비추천)**: **Root 사용자**(IAM 사용자 불가능)로 계정의 CloudFront Key Pair를 다운받아 활용하는 방식이다.
    - AWS Root 사용자를 활용해야 하며, API를 사용할 수 없고, IAM을 통한 권한 제어가 **불가능**하다.
- CloudFront URL을 만들 때 **Distribution** 단위로 등록된 **Key Group**을 활용한다.
  - **Key Group**: Private/Public Key로 이루어진 키 쌍의 집합으로, CloudFront에 업로드하는 하나 이상의 **Public Key**로 구성된다.

#### Presigned 정책 (Policy)

- Presigned URL/Cookie를 만들 때 URL/Cookie의 **권한**을 설정하기 위한 정책이다.
- **두 가지 종류**가 있다.
  - **Canned Policy(미리 준비된 정책)**: 간단한 버전으로, **만료 시간**만 설정 가능하다.
    - URL이 **짧아지는** 장점이 있다.
  - **Custom Policy**: 모든 제약 사항 설정이 가능하다.
    - 정책으로 Presigned URL/Cookie의 동작 범위(**만료 시간**, **시작 시간**, **IP 제한** 등) 설정이 가능하다.

| 설명                | Canned Policy | Custom Policy  |
| :------------------ | :-----------: | :------------: |
| 정책 재사용         |      No       |      Yes       |
| 사용 가능 시점 적용 |      No       | Yes (Optional) |
| 만료 시점 적용      |      Yes      |      Yes       |
| IP Range 제한       |      No       | Yes (Optional) |

### 7.2. Origin 접근 제한

#### OAC, OAI

- CloudFront를 통해서만 S3에 접근하도록 제한하여, 유저가 S3 URL로 직접 접속하는 것을 **차단**하는 기능이다.
- **OAI(Origin Access Identity)**
  - CloudFront만 S3의 파일에 접근할 수 있도록 설정하는 **기존(Legacy)** 안전장치이다.
  - CloudFront 배포에 특별한 **자격 증명**을 생성하고 이를 S3 버킷 정책에 허용하는 방식이다.
- **OAC(Origin Access Control)**
  - **최신 권장** 방식이며, **IAM**을 사용하여 더 세부적이고 유연한 보안 설정이 가능하다.
  - OAI의 모든 기능을 포함한다.
  - S3에서 해당 OAC의 접근을 허용하고, CloudFront에서 OAC를 활용해서 S3와 소통한다.
  - S3에서 기본적으로 모든 접근을 차단하고 OAC의 접근만 허용한다.
  - **Lambda Function URL**에도 사용 가능하다.

#### Origin Access Control의 Sign 방법

- CloudFront가 S3와 소통하기 위한 요청에 **Sign 방법**을 정의한다.
- 세 가지 종류
  - **Sign requests (recommended)**: CloudFront IAM Principal이 S3에 요청할 때 **SigV4**로 Sign한다.
    - 요청에 자격 증명을 활용해 필요한 정보로 **Authorization Header**를 구성하고, S3에서 해당 내용을 검증해서 자격이 있는 요청인지 확인 후 처리하거나 거부한다.
    - 클라이언트가 **Sign**한 헤더가 있다면 **덮어씌운다(Override)**.
  - **Do not override authorization header**: 클라이언트 Header가 있다면 사용하고, 없으면 새로 **Sign**한다.
  - **Do not sign requests**: **Authorization Header**를 사용하지 않는다.
    - 클라이언트가 항상 Sign을 통해 요청하거나, 콘텐츠가 **퍼블릭**인 경우 사용한다.

#### Custom Origin 보호

- **Custom Header**를 활용한다.
  - CloudFront에서 **Header**를 생성하고, Origin에서 해당 Header가 없으면 **거부**한다.
- Origin에서 **CloudFront IP**를 제외한 모든 트래픽을 **차단**한다.

#### 지리적 배포 제한 (Geographic Restrictions)

- **CloudFront 지리 배포 제한**
  - **Allow list** 혹은 **Block list**로 **국가 기준** 설정을 한다.
  - 모든 배포(**Distribution**)에 제한 사항이 포함된다.
    - 즉 일부만 제한을 거는 것은 **불가능**하다.
  - IP 주소의 정확도는 99.8%이다.
- **3rd Party 지리적 위치 서비스** 사용
  - **커스터마이징**이 가능하다(브라우저별, 쿠키별 등).
  - **Signed URL** 기반으로 동작한다.

#### Field-Level Encryption

- CloudFront를 활용해서 실제 데이터를 처리하는 주체까지 데이터를 **암호화**해서 전달할 수 있는 방법이다.
  - HTTPS 통신과는 별도의 개념이다.
- Edge Location에서 받은 데이터 중 특정 데이터를 주어진 **Public Key**로 암호화한다.
- 이후 데이터를 처리하는 측에서 **Private Key**로 **복호화**하여 사용한다.

## 8. Amazon CloudFront 모니터링

### 8.1. CloudWatch 지표

- CloudWatch를 통해서 다양한 **지표(Metric)** 를 제공한다.
  - **기본 지표**와 추가 비용으로 활성화 가능한 **추가 지표**로 나뉜다.
    - **기본 지표**: 요청 수, **BytesDownloaded**, **BytesUploaded**, **4xx**, **5xx**, **TotalErrorRate** 등이 있다.
    - **추가 지표**: **CacheHitRate**, **OriginLatency**, **ErrorRate** by Status Code(401, 403, 404, 502, 503, 504) 등이 있다.
  - 추가 지표 비용은 고정이며 활성화된 지표별로 한 달에 한 번 **배포(Distribution)** 당 발생한다.
- **버지니아 리전(us-east-1)** 에서 확인 가능하다.

### 8.2. Access Log

- **Access Log(Standard Log)**: S3 버킷을 지정해서 모든 유저의 요청을 **로깅**한다.
  - 실시간이 아니며 **지연 시간**이 발생한다.
- **Real-Time Log**: 약 **초 단위**의 지연 시간으로 요청을 **실시간**으로 로깅한다.

#### Access Log (Standard)

- S3 버킷을 지정해서 모든 유저의 요청을 **로깅**한다.
- **Distribution** 단위로 요청받은 Edge Location에서 지속적으로 로그 파일을 만들어 **S3 버킷**으로 **Flush**한다.
- 시간 단위로 여러 번 **Flush**한다.
  - 최대 **24시간**까지 지연 가능하며, 심지어 아예 전송되지 않고 **누락**될 수 있다.
  - 헤더와 쿠키의 크기가 **20KB**를 넘거나 URL이 **8,192Bytes**를 넘어갈 경우 CloudFront에서 요청을 별도로 **파싱(Parse)** 하지 않고 처리한다.
    - 이 경우 **로깅**이 되지 않는다.

#### Real-Time Log

- CloudFront의 요청 로그를 **실시간**으로 처리할 수 있는 기능이다.
  - **Kinesis Data Stream**으로 처리한다.
  - 추후 **Firehose** 등으로 S3에 로그로 저장하거나, **Redshift**, **OpenSearch** 등으로 분석 가능하다.
- **추가 비용**이 발생한다.
- 로그의 **지연 시간**이 발생하거나 **누락**될 수 있다.
- 설정 값
  - **Sampling Rate**: 요청 중 얼마만큼을 받아볼 것인지에 대한 설정이다(1~100%).
  - **Fields**: 로그 내용 중 실시간으로 받아볼 **필드**(Timestamp, Client IP, TimeToFirstByte, StatusCode, Host, EdgeLocation, TimeTaken 등)를 선택한다.
  - **Cache 동작**: 실시간 로그를 받아볼 **패턴** 단위의 동작(**Behavior**)을 지정한다.
- **Kinesis Stream** 권한 설정에 **IAM 역할**이 필요하다.
