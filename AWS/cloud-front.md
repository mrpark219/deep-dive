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

#### S3 Origin Access (OAI vs OAC)

- CloudFront를 통해서만 S3에 접근하도록 제한하여, 유저가 S3 URL로 직접 접속하는 것을 **차단**하는 기능이다.
- **OAI(Origin Access Identity)**
  - CloudFront만 S3의 파일에 접근할 수 있도록 설정하는 **기존(Legacy)** 안전장치이다.
  - CloudFront 배포에 특별한 **자격 증명**을 생성하고 이를 S3 버킷 정책에 허용하는 방식이다.
- **OAC(Origin Access Control)**
  - **최신 권장** 방식이며, **IAM**을 사용하여 더 세부적이고 유연한 보안 설정이 가능하다.
  - OAI의 모든 기능을 포함한다.

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
