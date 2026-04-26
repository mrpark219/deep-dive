# HTTP 서버 만들기

## 1. AutoFlush

```java
new PrintWriter(socket.getOutputStream(), false, UTF_8)
```

- **`PrintWriter`** 객체를 생성할 때 두 번째 인자는 **`autoFlush`**(자동 플러시) 여부를 결정한다.
- 이 값을 **`true`** 로 설정하면 `println()`으로 출력할 때마다 자동으로 플러시되어 첫 내용을 빠르게 전송할 수 있지만, 네트워크 전송(패킷 발생)이 너무 자주 일어난다는 단점이 있다.
- 이 값을 **`false`** 로 설정하면 **`flush()`** 를 직접 호출해야만 데이터를 전송하며, 데이터를 모아서 한 패킷에 담아 전송하므로 네트워크 전송 횟수를 효과적으로 줄일 수 있다.
- 위 코드에서는 `false`로 설정했으므로, 데이터 작성이 끝나면 마지막에 반드시 **`writer.flush()`** 를 호출해야 한다.

## 2. requestToString

```java
private String requestToString(BufferedReader reader) throws IOException {
    StringBuilder sb = new StringBuilder();
    String line;

    while ((line = reader.readLine()) != null) {
        if (line.isEmpty()) {
            break;
        }
        sb.append(line).append("\n");
    }

    return sb.toString();
}
```

- 클라이언트가 보낸 HTTP 요청(Request)을 읽어서 **`String`** 으로 반환한다.
- HTTP 요청 메시지의 시작 라인과 헤더까지만 우선적으로 읽어 들인다.
- 문자열을 읽다가 **`line.isEmpty()`**(빈 라인)를 만나면, 이를 HTTP 메시지 헤더의 마지막으로 인식하고 메시지 읽기를 종료한다.
- HTTP 메시지는 **빈 라인(Empty Line)** 을 기준으로 헤더(Header)와 바디(Body)를 구분하기 때문이다.

## 3. ResponseToClient

```java
private void responseToClient(PrintWriter writer) {
    String body = "<h1>Hello World</h1>";
    int length = body.getBytes(UTF_8).length;

    StringBuilder sb = new StringBuilder();
    sb.append("HTTP/1.1 200 OK\r\n");
    sb.append("Content-Type: text/html; charset=utf-8\r\n");
    sb.append("Content-Length: ").append(length).append("\r\n");
    sb.append("\r\n"); // header, body 구분 라인
    sb.append(body);

    writer.println(sb);
    writer.flush();
}
```

- HTTP 응답(Response) 메시지를 생성하여 클라이언트에게 다시 전달한다.
- 응답 메시지는 시작 라인, 헤더, 그리고 실제 데이터가 담긴 HTTP 메시지 바디로 구성하여 전달한다.
- HTTP 공식 스펙에서 줄 바꿈 기호는 **`\r\n` (캐리지 리턴 + 라인 피드)** 로 표현해야 원칙에 맞다. (단, `\n`만 사용해도 대부분의 현대 웹 브라우저들은 문제없이 처리해 준다.)
- 응답 메시지 생성이 모두 끝나면 마지막에 반드시 **`writer.flush()`** 를 호출해서 실제 데이터를 네트워크로 전송해야 한다.

## 4. HTTP 요청 메시지

### 4.1. 시작 라인

- **`GET`**: HTTP 요청 메서드로, 서버에 리소스 조회를 요청할 때 사용한다.
- **`/`**: 요청 경로(Path)를 나타내며, 별도의 특정 경로를 지정하지 않으면 루트(`/`)를 사용한다.
- **`HTTP/1.1`**: 통신에 사용하는 HTTP 버전을 나타낸다.

### 4.2. 헤더

- **`Host`**: 클라이언트가 접속하려는 목적지 서버의 도메인 정보이다.
- **`User-Agent`**: 요청을 보내는 주체인 웹 브라우저나 클라이언트 애플리케이션의 정보이다.
- **`Accept`**: 웹 브라우저가 전달받을 수 있는 HTTP 응답 메시지 바디의 데이터 형태(MIME 타입)를 지정한다.
- **`Accept-Encoding`**: 웹 브라우저가 처리할 수 있는 데이터 압축(인코딩) 형태를 서버에 알려준다.
- **`Accept-Language`**: 웹 브라우저가 선호하는 자연어 형태(한국어, 영어 등)를 지정한다.

## 5. HTTP 응답 메시지

### 5.1. 시작 라인

- **`HTTP/1.1`**: 서버가 응답에 사용하는 HTTP 버전을 나타낸다.
- **`200`**: 클라이언트의 요청이 정상적으로 처리되었음을 알리는 HTTP 상태 코드(성공)이다.
- **`OK`**: 상태 코드 `200`에 대해 사람이 읽을 수 있도록 덧붙이는 설명 문구(Reason Phrase)이다.

### 5.2. 헤더

- **`Content-Type`**: HTTP 메시지 바디에 담긴 데이터의 형식(예: HTML)을 서버가 클라이언트에게 알려주어 올바르게 파싱하도록 한다.
- **`Content-Length`**: HTTP 메시지 바디의 정확한 데이터 크기를 바이트(Byte) 단위로 나타낸다.

### 5.3. 바디

- **`<h1>Hello World</h1>`**: 실제 클라이언트 화면에 렌더링될 데이터 내용(HTML 등)이 들어간다.

## 6. URL 인코딩

- HTTP 메시지에서 시작 라인(URL 포함)과 HTTP 헤더의 이름은 반드시 **ASCII** 문자열만 사용해야 한다.
- HTTP 메시지 바디의 경우 **UTF-8**과 같은 다양한 인코딩 방식을 자유롭게 사용할 수 있다.
- 인터넷이 처음 설계되던 1980~1990년대의 대부분 컴퓨터 시스템은 기본적으로 **ASCII 문자 집합**을 사용했다.
- 전 세계의 다양한 컴퓨터 시스템과 네트워크 장비 간의 완벽한 **호환성**을 보장하기 위해 URL은 단일한 문자 인코딩 체계(ASCII)를 채택해야만 했다.
- 순수한 UTF-8로 URL을 표현하려면 전 세계의 모든 장비가 이를 완벽히 지원해야 하지만, 여전히 수많은 시스템이 ASCII 기반 표준에 의존하므로 심각한 **호환성 문제**가 발생할 수 있다.

### 6.1. 퍼센트(%) 인코딩

- 한글을 UTF-8 인코딩으로 표현하면 한 글자당 **3byte**의 데이터를 차지하게 된다.
- URL은 ASCII 문자만 허용하므로, 이러한 3byte의 UTF-8 문자를 그대로 네트워크에 전송할 수 없다.
- 예를 들어 한글 '가'를 UTF-8 16진수로 변환하면 `EA`, `B0`, `80`이 되는데, 각각의 바이트 문자 앞에 **`%` (퍼센트)** 기호를 붙여서 표현한다.
- 각각의 16진수 byte를 문자로 표현하고 그 앞에 `%`를 붙이는 방식을 **퍼센트 인코딩(Percent Encoding)** 이라고 부른다.
- 퍼센트 인코딩은 데이터 크기 측면에서는 효율이 다소 떨어지지만, 네트워크 통신에서 가장 중요한 **보수적인 호환성**을 완벽하게 보장해 준다.
- 클라이언트가 문자 '가'를 전송하기 위해 먼저 문자를 UTF-8로 인코딩하여 `EA`, `B0`, `80`이라는 3byte를 획득한다.
- 획득한 각 byte를 16진수 문자로 표현하고 앞에 `%`를 붙여 **`%EA%B0%80`** 형태로 서버에 전송(`q=%EA%B0%80`)한다.
- 서버는 `%EA%B0%80`이라는 ASCII 문자를 전달받고, `%`가 붙은 것을 확인하여 디코딩해야 하는 문자로 인식한다.
- 서버는 퍼센트를 제거하고 `EA`, `B0`, `80` byte로 다시 변환한 뒤, 이를 UTF-8로 디코딩하여 최종적으로 원본 문자 '가'를 획득한다.

### 6.2. 자바에서 % 인코딩

```java
String encode = URLEncoder.encode("가", UTF_8);
System.out.println("encode = " + encode);

String decode = URLDecoder.decode(encode, UTF_8);
System.out.println("decode = " + decode);
```

- 자바에서는 기본적으로 제공하는 **`URLEncoder.encode()`** 와 **`URLDecoder.decode()`** 메서드를 사용하면 복잡한 퍼센트 인코딩과 디코딩 과정을 아주 쉽게 처리할 수 있다.
