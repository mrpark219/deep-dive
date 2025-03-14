# HTTP 메서드 활용

## 1. 클라이언트에서 서버로 데이터 전송

### 1.1 쿼리 파라미터(Query Parameter) 전송

- **HTTP GET 요청에 사용된다**.
- **주로 정렬, 필터, 검색어 등의 정보를 전달할 때 사용한다**.
- 예제:
  ```bash
  GET /members?sort=asc&page=2
  ```

### 1.2 메시지 바디(Body) 전송

- **POST, PUT, PATCH 요청에 사용된다**.
- **회원 가입, 상품 주문, 리소스 등록 및 변경 시 활용한다**.
- 예제:
  ```json
  POST /members
  {
  "name": "Test",
  "email": "Test@example.com"
  }
  ```

## 2. 데이터 전송 방식별 예시

### 2.1 정적 데이터 조회

- 정적 리소스(이미지, 정적 텍스트 문서 등)를 요청할 때 사용한다.
- GET 요청을 사용한다.
- 일반적으로 쿼리 파라미터 없이 리소스 경로로 조회할 수 있다.
- 예제:
  ```bash
  GET /images/logo.png
  ```

### 2.2 동적 데이터 조회

- 조회 조건을 줄여주는 필터, 조회 결과를 정렬하는 정렬 조건에 주로 사용한다.
- GET 요청을 사용하며 쿼리 파라미터(Query Parameter)를 통해 데이터를 전달한다.
- 예제:
  ```bash
  GET /products?category=shoes&sort=price_desc
  ```

### 2.3 HTML Form 데이터 전송

- HTML Form은 GET, POST 전송만 지원한다.
- GET 전송
  - 다중 검색 조건을 서버로 전송할 때 사용한다.
- POST 전송
  - 회원 가입, 상품 주문, 데이터 변경 등을 요청할 때 사용한다.
  - Content-Type: `application/x-www-form-urlencoded`
    - Form 데이터를 key=value 형식으로 전송한다. (URL encoding 포함)
- Content-Type: `multipart/form-data`
  - 파일 업로드 같은 바이너리 데이터 전송 시 사용한다.
  - 다른 종류의 여러 파일과 form의 내용을 함께 전송할 수 있다.
- 예제:
  ```bash
  POST /members
  Content-Type: application/x-www-form-urlencoded
  Body: name=John&email=john@example.com
  ```

### 2.4 HTTP API 데이터 전송

- 백엔드 시스템 간 통신, 웹 클라이언트(AJAX) 요청, 모바일 앱 통신 등에서 사용한다.
- POST, PUT, PATCH → 메시지 바디로 데이터를 전송한다.
- GET → 조회 시 쿼리 파라미터를 사용한다.
- Content-Type: `application/json`를 주로 사용한다.
- 예제:
  ```json
  POST /members
  Content-Type: application/json
  {
    "name": "John",
    "email": "john@example.com"
  }
  ```

## 3. HTTP API 설계 예시

### 3.1 회원 관리 시스템 (POST 기반 등록)

| 기능           | HTTP 메서드      | URI             |
| -------------- | ---------------- | --------------- |
| 회원 목록 조회 | GET              | `/members`      |
| 회원 등록      | POST             | `/members`      |
| 회원 조회      | GET              | `/members/{id}` |
| 회원 수정      | PUT, PATCH, POST | `/members/{id}` |
| 회원 삭제      | DELETE           | `/members/{id}` |

- 클라이언트는 등록될 리소스의 URI를 모른다.
- **서버가 새로 등록된 리소스 URI를 생성**해준다.
- **컬렉션(Collection)**
  - 서버가 관리하는 리소스 디렉토리이다.
  - 서버가 리소스의 URI를 생성하고 관리한다.
  - 여기서 컬렉션은 `/members`

### 3.2 파일 관리 시스템 (PUT 기반 등록)

| 기능           | HTTP 메서드 | URI                 |
| -------------- | ----------- | ------------------- |
| 파일 목록 조회 | GET         | `/files`            |
| 파일 조회      | GET         | `/files/{filename}` |
| 파일 등록      | PUT         | `/files/{filename}` |
| 파일 삭제      | DELETE      | `/files/{filename}` |
| 파일 대량 등록 | POST        | `/files`            |

- 클라이언트가 리소스 URI를 알고 있어야 한다.
- **클라이언트가 직접 리소스의 URI를 지정**한다.
- **스토어(Store)**
  - 클라이언트가 관리하는 리소스 저장소이다.
  - 클라이언트가 리소스의 URI를 알고 관리한다.
  - 여기서 스토어는 `/files`

### 3.3 HTML Form을 사용하는 회원 관리

| 기능              | HTTP 메서드 | URI                    |
| ----------------- | ----------- | ---------------------- |
| 회원 목록 조회    | GET         | `/members`             |
| 회원 등록 폼 조회 | GET         | `/members/new`         |
| 회원 등록         | POST        | `/members`             |
| 회원 조회         | GET         | `/members/{id}`        |
| 회원 수정 폼 조회 | GET         | `/members/{id}/edit`   |
| 회원 수정         | POST        | `/members/{id}`        |
| 회원 삭제         | POST        | `/members/{id}/delete` |

- **컨트롤 URI**
  - 동사 기반의 URI 사용 (`/new`, `/edit`, `/delete`)
  - HTTP 메서드로 해결하기 애매한 경우 사용
    - HTML FORM은 GET, POST만 지원

## 4. URI 설계 개념

### 4.1 문서(Document)

- **단일 개념이다. (파일 하나, DB 레코드 등)**
- **예제**: `GET /members/100`

### 4.2 컬렉션(Collection)

- **서버가 관리하는 리소스 디렉터리이다**.
- **서버가 리소스 URI를 생성하고 관리한다**.
- **예제**: `GET /members`

### 4.3 스토어(Store)

- **클라이언트가 직접 리소스 URI를 지정하고 관리한다**.
- **예제**: `PUT /files/myfile.jpg`

### 4.4 컨트롤러(Controller), 컨드롤 URI

- **문서, 컬렉션, 스토어 패턴으로 해결하기 어려운 추가 프로세스를 실행한다**.
- **동사(Verb)를 포함하는 URI를 의미한다**.
- **예제**: `POST /members/{id}/delete`
