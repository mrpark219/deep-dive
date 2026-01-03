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
