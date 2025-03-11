# URI와 웹 브라우저 요청 흐름

## 1. URI

### 1.1 URI, URL, URN의 차이점

- **URI(Uniform Resource Identifier)**: 리소스를 식별하는 방식이다.
- **URL(Uniform Resource Locator)**: 리소스의 위치를 나타낸다.
- **URN(Uniform Resource Name)**: 리소스의 이름을 나타낸다.
- URI는 로케이터(Locator), 이름(Name) 또는 둘 다 포함할 수 있다.
- URI는 **URL과 URN을 포함하는 개념**이다.

### 1.2 URI의 구성 요소

- **Uniform**: 리소스를 식별하는 통일된 방식이다.
- **Resource**: URI로 식별할 수 있는 모든 자원(제한 없음)이다.
- **Identifier**: 다른 항목과 구분하는 데 필요한 정보이다.

### 1.3 URL과 URN

- **Locator(URL)**: 리소스가 어디에 있는지 위치를 지정하는 역할을 한다.
- **Name(URN)**: 리소스의 이름을 부여하는 역할을 한다.
- **위치는 변할 수 있지만, 이름은 변하지 않는다.**
- **URN만으로 리소스를 찾는 방법은 아직 보편화되지 않았다.**
- **실무에서는 대부분 URI = URL로 사용한다.**

---

## 2. URL의 구조

### 2.1 URL의 구성 요소

- `scheme://[userinfo@]host[:port][/path][?query][#fragment]`
  <br>
- **scheme**: 프로토콜을 지정한다. (예: `http`, `https`)
- **userinfo**: URL에 사용자 정보를 포함하여 인증하는데 사용되며, 거의 사용하지 않는다.
- **host**: 도메인 이름을 지정한다. (예: `www.example.com`)
- **port**: 포트 번호를 지정하며, 기본 포트일 경우 생략 가능하다. (예: `80`, `443`)
- **path**: 리소스 경로를 계층적 구조로 표현한다. (예: `/test`)
- **query**: 쿼리 파라미터를 key=value 형태로 지정하며, `?`로 시작하고 `&`로 여러 개를 추가할 수 있다. (예: `word=hello-world&test=hi`)
- **fragment**: HTML 내부 북마크 등에 사용되며, 서버로 전송되지 않는다. (예: `#content`)

---

## 3. 웹 브라우저의 요청 흐름

### 3.1 웹 브라우저 요청의 전체 과정

1. 사용자가 웹 브라우저에 `https://www.example.com:443/search?q=hello`를 입력한다.
2. **DNS 서버에 www.example.com 도메인을 조회하여 서버의 IP 주소를 찾는다.**
3. **찾은 서버의 IP 주소와 포트 정보를 조합하여 웹 브라우저가 HTTP 요청 메시지를 생성한다.**
4. **SOCKET 라이브러리를 통해 TCP/IP 계층에 요청을 전달한다.**
5. **TCP 3-way-handshake를 통해 서버와 연결을 설정한다.**
6. **TCP/IP 패킷을 생성하여 서버로 전달하며, 이때 HTTP 메시지를 데이터로 포함한다.**
7. **서버가 요청을 수신하고, HTTP 요청 메시지를 해석하여 처리한다.**
8. **서버가 HTTP 응답 메시지를 생성하고 클라이언트(웹 브라우저)로 전송한다.**
9. **웹 브라우저가 응답 메시지를 해석하여 사용자에게 화면을 표시한다.**
