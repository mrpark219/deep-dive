# 스트림

## 1. 스트림

- 자바가 가진 데이터를 파일(`hello.dat` 등)에 저장하려면 데이터의 통로 역할을 하는 **스트림(Stream)** 이 필요하다.
- 프로세스 내부의 데이터를 외부로 내보낼 때는 **출력 스트림(Output Stream)** 을 사용하고, 반대로 외부 데이터를 내부로 가져올 때는 **입력 스트림(Input Stream)** 을 사용한다.
- 데이터는 항상 한쪽으로만 이동하기 때문에 각 스트림은 **단방향**으로 흐른다.

## 2. 자바 스트림 함수

### 2.1. new FileOutputStream("temp/hello.dat")

- 파일에 데이터를 기록하기 위한 **출력 스트림**이다.
- 지정한 경로에 파일이 없으면 **자동으로 생성**하여 데이터를 저장한다.
- 단, 폴더 자체를 만들어주지는 않으므로 **사전에 폴더가 존재**해야 한다.

#### append 옵션

- `new FileOutputStream("temp/hello.dat", true)`
- 두 번째 인자에 `true`를 전달하면 기존 파일 끝에 데이터를 **이어서 작성**한다.
- `false`를 전달하거나 생략하면 기존 데이터를 모두 지우고 **처음부터 다시 작성**하며, 이것이 **기본값**이다.

### 2.2. write()

- 데이터를 **1바이트(byte) 단위**로 파일에 출력한다.

#### write(byte[])

- 원하는 데이터를 `byte[]` 배열에 담아 `write()` 메서드에 전달하면 여러 데이터를 **한 번에 출력할 수 있다**.

### 2.3. new FileInputStream("temp/hello.dat")

- 지정한 파일에서 데이터를 읽어오기 위한 **입력 스트림**이다.

### 2.4. read()

- 파일에서 데이터를 **1바이트(byte) 단위**로 하나씩 읽어온다.
- 더 이상 읽을 내용이 없는 파일의 끝(**EOF**, End of File)에 도달하면 **`-1`을 반환**한다.

#### 파일을 읽었을 때 ABC가 나오는 이유

- `hello.dat` 파일에 65, 66, 67을 저장하면 이 값들은 **바이트(byte)** 단위의 데이터로 변환되어 저장된다.
- 텍스트 편집기나 개발 도구에서 이 파일을 열 때는 **UTF-8**이나 **MS949**와 같은 문자 집합을 사용하여 바이트 데이터를 문자로 **디코딩**한다.
- 결과적으로 이러한 문자 변환 과정을 거치기 때문에 사용자 화면에는 숫자가 아닌 **ABC**가 출력되는 것이다.

#### read()가 int를 반환하는 이유

- **부호 없는 바이트를 온전히 표현해야 하기 때문**이다.
  - 자바에서 `byte`는 `-128 ~ 127` 범위를 가지는 **부호 있는 8비트 값**이다.
  - 반환형을 `int`로 설정하면 `0 ~ 255`까지의 모든 바이트 값을 **부호 없이 온전하게 표현**할 수 있다.
- **스트림의 끝(EOF, End of File)을 명확하게 표시하기 위해서**이다.
  - 바이트 데이터를 온전히 읽어오려면 256가지의 숫자 값을 모두 사용해야 한다.
  - 자바의 `byte`는 이 256가지 값(-128 ~ 127)을 데이터 표현에 전부 써버리기 때문에, 더 이상 읽을 데이터가 없다는 것을 의미하는 **특별한 값을 할당할 공간이 없다**.
  - 하지만 `int`를 반환형으로 사용하면 `0 ~ 255`로 실제 데이터를 표현하고, 추가로 **`-1`을 반환하여 스트림의 끝(EOF)을 나타낼 수 있다**.
- 데이터를 출력하는 **`write()` 메서드 역시 같은 이유로 `int` 타입을 입력받는다**.

#### read(byte[], offset, length)

- 미리 생성해 둔 `byte[]` 배열을 활용하여 데이터를 **한 번에 읽어올 수 있다**.
- `byte[]`: 데이터를 임시로 담아두는 **버퍼(Buffer) 역할**을 한다.
- `offset`: 읽어온 데이터를 배열의 어느 위치부터 저장할지 지정하는 **시작 인덱스**이다.
- `length`: 한 번에 읽어올 수 있는 **최대 바이트 길이**이다.
- **반환 값**: 버퍼에 실제로 읽어 들인 **총 바이트 수**를 반환한다.
  - 예를 들어 3바이트를 읽어왔다면 **3을 반환한다**.
  - 스트림의 끝(EOF)에 도달하여 더 이상 읽을 데이터가 없으면 **`-1`을 반환한다**.

#### 부분으로 나누어 읽기 vs 전체 읽기

- `read(byte[], offset, length)`를 활용하여 데이터를 **부분적으로 읽는 방식이다**.
  - 스트림의 내용을 일부분씩 나누어 읽거나, 데이터를 읽는 즉시 처리해야 할 때 **적합하다**.
  - 가장 큰 장점은 프로그램의 **메모리 사용량을 직접 제어할 수 있다는 것**이다.
  - 대용량 파일을 처리할 때 한 번에 메모리에 올리지 않고 **조각내어 읽어 들일 때 유용하다**.
  - 예를 들어 100MB 크기의 파일을 1MB 단위로 나누어 읽으면, 시스템은 한 번에 **최대 1MB의 메모리만 사용한다**.
- `readAllBytes()`를 활용하여 전체 데이터를 **한 번에 읽는 방식이다**.
  - 메서드를 단 한 번만 호출해도 파일의 모든 데이터를 읽어올 수 있어 **매우 편리하다**.
  - 파일 크기가 작거나, 메모리에 모든 내용을 올려두고 한 번에 처리해야 하는 상황에 **적합하다**.
  - 단점은 한 번에 모든 데이터를 가져오므로 **메모리 사용량을 제어할 수 없다는 것**이다.
  - 용량이 큰 대용량 파일을 이 방식으로 읽어 들이면 메모리가 부족해져서 **`OutOfMemoryError`가 발생할 수 있다**.

### 2.5. close()

- 자바의 내부 객체는 가비지 컬렉터(GC)가 알아서 정리하지만, 파일과 같은 **외부 자원**은 직접 닫아주어야 한다.
- 스트림 사용이 끝나면 메모리 누수를 막기 위해 반드시 **`close()`를 호출하여 자원을 해제**해야 한다.

## 3. InputStream, OutputStream

- 자바 내부의 데이터를 외부에 있는 파일에 저장하거나, 네트워크를 통해 전송하거나, 콘솔에 출력할 때는 모두 **바이트(byte) 단위**로 데이터를 주고받는다.
- 파일, 네트워크, 콘솔마다 데이터를 주고받는 방식이 제각각이라면 개발하기 **매우 불편할 것이다**.
- 이러한 문제를 해결하기 위해 자바는 `InputStream`과 `OutputStream`이라는 **기본 추상 클래스**를 제공한다.
  - 일부 작동하는 코드도 있기 때문에 인터페이스가 아니라 추상 클래스도 제공된다.
- 따라서 스트림을 활용하면 파일 입출력이나 소켓을 통한 네트워크 통신 등 어떤 상황에서도 **일관된 방식**으로 데이터를 주고받을 수 있다.
  - 이를 위해 자바는 수많은 **기본 구현 클래스**들을 함께 제공한다.
  - 필요에 따라 개발자가 직접 **커스텀 기능**을 추가할 수도 있다.

### 3.1. InputStream

- 데이터를 읽어 들이는 입력 스트림이며, `read()`, `read(byte[])`, `readAllBytes()` 등의 메서드를 제공한다.
- 주요 구현 클래스에는 `FileInputStream`, `ByteArrayInputStream`, `SocketInputStream` 등이 있다.

### 3.2. OutputStream

- 데이터를 내보내는 출력 스트림이며, `write(int)`, `write(byte[])` 등의 메서드를 제공한다.
- 주요 구현 클래스에는 `FileOutputStream`, `ByteArrayOutputStream`, `SocketOutputStream` 등이 있다.

## 4. 파일 입출력과 성능 최적화

```java
FileOutputStream fos = new FileOutputStream("test.dat");
for (int i = 0; i < 10 * 1024 * 1024; i++) {
    fos.write(1);
}
fos.close();

FileInputStream fis = new FileInputStream("test.dat");
int fileSize = 0;
int data;
while ((data = fis.read()) != -1) {
    fileSize++;
}
fis.close();
```

- 위 코드는 10MB 크기의 파일을 1바이트씩 쓰고 읽는 과정을 **보여준다**.
- 10MB를 쓰는 데 약 15초, 읽는 데 약 5초라는 **매우 오랜 시간이 걸린다**.
  - 이렇게 오래 걸리는 이유는 자바 프로그램이 디스크에 데이터를 **1바이트(byte)씩 전달하기 때문**이다.
  - 디스크가 1바이트의 데이터를 받아서 1바이트씩 기록하는 것이다.
  - 결국 이 과정을 무려 1000만 번이나 반복하게 된다.
- 성능 저하의 원인은 더 자세히 살펴보면 **두 가지 이유**로 설명할 수 있다.
  - 첫째, `write()`나 `read()`를 호출할 때마다 운영체제(OS)의 **시스템 콜(System Call)** 을 통해 파일을 읽고 쓰라는 명령을 전달한다.
  - 이러한 시스템 콜은 컴퓨터 자원을 많이 소모하는 상대적으로 **무거운 작업**이다.
  - 둘째, HDD나 **SSD** 같은 저장 장치들도 데이터를 한 번 읽고 쓸 때마다 기본적으로 **필요한 시간**이 존재한다.
  - 특히 HDD는 디스크를 물리적으로 회전시켜야 하므로 훨씬 더 느리다.
- 다행히 자바에서 1바이트씩 전달하더라도, 운영체제나 하드웨어 레벨에서 자체적인 **성능 최적화가 발생한다**.
  - 따라서 실제로 디스크에 1바이트씩 곧이곧대로 **쓰는 것은 아니다**.
  - 하지만 어느 정도 최적화가 이루어지더라도, 너무 자주 발생하는 시스템 콜로 인한 성능 저하는 피할 수 없다.
  - 따라서 자바 애플리케이션 단에서 `read()`와 `write()`의 **호출 횟수 자체를 줄여야 한다**.

### 4.1. 버퍼 활용

```java
byte[] buffer = new byte[BUFFERED_SIZE];
int bufferIndex = 0;

for (int i = 0; i < FILE_SIZE; i++) {
    buffer[bufferIndex++] = 1;

    // 버퍼가 가득 차면 데이터를 디스크에 쓰고, 버퍼의 인덱스를 초기화한다.
    if (bufferIndex == BUFFERED_SIZE) {
        fos.write(buffer);
        bufferIndex = 0;
    }
}

// 반복문이 끝난 후 버퍼가 가득 차지 않고 남아있는 자투리 데이터를 마저 쓴다.
if (bufferIndex > 0) {
    fos.write(buffer, 0, bufferIndex);
}
fos.close();

// 읽기 과정에서도 버퍼를 활용한다.
byte[] readBuffer = new byte[BUFFERED_SIZE];
int fileSize = 0;
int size;
while ((size = fis.read(readBuffer)) != -1) {
    fileSize += size;
}
fis.close();
```

- 앞서 언급한 1바이트 단위 입출력의 성능 저하 문제를 해결할 때 사용하는 것이 바로 **버퍼(Buffer)** 이다.
- 데이터를 한 번에 모아서 전달하거나, 모아서 전달받는 용도로 사용하는 임시 메모리 공간을 **버퍼라고 한다**.

#### 버퍼 크기에 따른 쓰기 성능

- 버퍼를 사용해 많은 데이터를 한 번에 묶어 전달하면 입출력 성능을 **최적화할 수 있다**.
- 운영체제에 작업을 요청하는 시스템 콜(System Call) 횟수가 줄어들며, HDD나 SSD 같은 장치의 물리적인 작동 횟수도 함께 **감소한다**.
- 예를 들어 입출력 단위를 1바이트에서 2바이트로만 변경해도 시스템 콜 횟수는 **절반으로 줄어든다**.
- 하지만 버퍼의 크기를 무작정 키운다고 해서 작업 속도가 계속해서 **빨라지는 것은 아니다**.
- 디스크나 파일 시스템이 데이터를 읽고 쓰는 기본 단위가 보통 **4KB 또는 8KB이기 때문**이다.
  - 4KB는 **4096바이트**이다.
  - 8KB는 **8192바이트**이다.
- 결과적으로 버퍼에 너무 많은 데이터를 담아서 보내도, 결국 디스크 내부에서는 해당 기본 단위로 잘게 나누어 저장하기 때문에 성능 향상에는 **명확한 한계가 있다**.
- 따라서 자바에서 버퍼의 크기를 설정할 때는 운영체제의 기본 입출력 단위에 맞춰 **4KB(4096)나 8KB(8192) 정도로 잡는 것이 가장 효율적**이다.

### 4.2. Buffered 스트림 활용

```java
// 1. BufferedOutputStream 활용 (쓰기)
FileOutputStream fos = new FileOutputStream(FILE_NAME);
BufferedOutputStream bos = new BufferedOutputStream(fos, BUFFERED_SIZE);

for (int i = 0; i < FILE_SIZE; i++) {
    bos.write(1);
}
bos.close();

// 2. BufferedInputStream 활용 (읽기)
FileInputStream fis = new FileInputStream(FILE_NAME);
BufferedInputStream bis = new BufferedInputStream(fis, BUFFERED_SIZE);

int fileSize = 0;
int data;
while ((data = bis.read()) != -1) {
    fileSize++;
}
bis.close();
```

- `BufferedOutputStream`과 `BufferedInputStream`은 내부적으로 데이터를 모아두는 **버퍼 기능**만 제공한다.
- 단독으로 사용할 수 없으므로 데이터를 실제로 입출력할 **대상 기본 스트림**(`FileOutputStream`, `FileInputStream` 등)이 반드시 필요하다.
- 객체를 생성할 때 대상 스트림과 함께 사용할 **버퍼의 크기**를 지정할 수 있다.
- 버퍼용 배열(`byte[]`)을 직접 다루지 않아도 되므로 코드를 **단순하게 작성**할 수 있다.
- `BufferedInputStream`은 `InputStream`을 상속받으므로 기존의 `read()` 메서드 등을 **그대로 사용**할 수 있다.

#### Buffered 스트림 실행 순서 - 쓰기

1. **내부 버퍼에 데이터 보관**
   - `write()` 메서드를 호출하면 데이터가 바로 디스크로 가지 않고 내부 버퍼 배열(`byte[] buf`)에 차곡차곡 보관된다.
2. **버퍼 가득 참 및 시스템 콜 발생**
   - 버퍼가 꽉 차면 대상 스트림의 `write(byte[])` 메서드를 **호출한다**.
   - 이때 **시스템 콜**을 통해 버퍼에 쌓인 모든 데이터를 한 번에 전달하고 버퍼를 **비운다**.
3. **잔류 데이터 처리 (Flush)**
   - 버퍼가 다 차지 않았더라도 `flush()`를 호출하면 현재 버퍼에 있는 데이터를 즉시 목적지로 **전송한다**.

#### Buffered 스트림 실행 순서 - 읽기

1. **내부 버퍼 확인 및 데이터 로드**
   - `read()` 메서드를 호출할 때 버퍼가 비어있다면 대상 스트림의 `read(byte[])`를 **호출한다**.
   - 디스크나 외부 소스에서 버퍼 크기만큼의 데이터를 한 번에 읽어와 버퍼를 **채운다**.
2. **버퍼에서 데이터 반환**
   - 이후 `read()`를 호출하면 디스크에 직접 접근하지 **않는다**.
   - 메모리 내 버퍼에 이미 보관된 데이터를 하나씩 꺼내어 매우 빠르게 **반환한다**.
3. **반복 및 재충전**
   - 버퍼의 데이터를 모두 소진하면 다시 1번 과정으로 돌아가 다음 뭉치 데이터를 버퍼에 새로 **채운다**.

#### 자원 정리 (close)

- 입출력 작업이 끝나면 메모리 누수 방지와 외부 자원 해제를 위해 반드시 `close()`를 **호출해야 한다**.
- 스트림을 여러 개 연결해서 사용할 때는 항상 가장 마지막에 감싼 **보조 스트림**만 닫아주면, 연결된 기본 스트림의 `close()`까지 연쇄적으로 **호출되어 정리된다**.
- 특히 쓰기 작업(`BufferedOutputStream`)에서는 자원을 닫기 전에 내부적으로 `flush()`가 먼저 실행되어 버퍼에 남은 자투리 데이터를 목적지에 **안전하게 저장한다**.
- 만약 보조 스트림을 닫지 않고 대상 스트림만 직접 닫으면, 쓰기 작업 시 버퍼 내부에 있던 데이터가 디스크에 쓰이지 않고 증발하는 **심각한 버그**가 생길 수 있다.

#### 기본 스트림과 보조 스트림

- `FileOutputStream`처럼 단독으로 생성하여 사용할 수 있는 스트림을 **기본 스트림**이라고 한다.
- `BufferedOutputStream`처럼 단독으로 사용할 수 없고 다른 스트림에 부가적인 기능을 제공하는 스트림을 **보조 스트림**이라고 한다.
- 보조 스트림 객체를 생성할 때는 반드시 **대상 기본 스트림**이 필요하다.

#### 버퍼를 직접 다루는 것보다 BufferedXxx의 성능이 조금 떨어지는 이유

```java
@Override
public void write(int b) throws IOException {
    if (lock != null) {
        lock.lock();
        try {
            implWrite(b);
        } finally {
            lock.unlock();
        }
    } else {
        synchronized (this) {
            implWrite(b);
        }
    }
}

public int read() throws IOException {
    if (lock != null) {
        lock.lock();
        try {
            return implRead();
        } finally {
            lock.unlock();
        }
    } else {
        synchronized (this) {
            return implRead();
        }
    }
}
```

- 배열(`byte[]`)로 버퍼를 직접 구현하는 방식과 `BufferedXxx` 클래스를 사용하는 방식은 둘 다 버퍼를 활용하므로 이론상 **비슷한 성능**이 나와야 한다.
- 하지만 실제로는 버퍼를 직접 다루는 방식이 조금 더 빠른데, 그 이유는 바로 내부의 **동기화(Synchronization) 처리** 때문이다.
- 데이터를 1바이트씩 읽고 쓸 때마다 스레드에 락(Lock)을 걸고 푸는 동기화 코드가 **반복적으로 실행**된다.
- 이 클래스들은 초기 자바의 **멀티 스레드 환경**을 고려하여 안전하게 설계되었으나, 단일 스레드(싱글 스레드) 환경에서는 불필요한 락 처리로 인해 **성능이 약간 저하**될 수 있다.
- 성능 최적화가 극도로 중요하다면, 내장 클래스에 의존하지 않고 기존 코드를 참고하여 동기화 블록을 제거한 **커스텀 클래스를 직접 만들어 사용**해야 한다.

### 4.3. 한 번에 쓰고 읽기

```java
FileOutputStream fos = new FileOutputStream(FILE_NAME);

byte[] buffer = new byte[FILE_SIZE];
for (int i = 0; i < FILE_SIZE; i++) {
    buffer[i] = 1;
}
fos.write(buffer);
fos.close();

FileInputStream fis = new FileInputStream(FILE_NAME);

byte[] bytes = fis.readAllBytes();
fis.close();
```

- 파일의 크기가 크지 않다면 데이터를 **한 번에 쓰고 읽는 것도 좋은 방법**이다.
- 입출력 속도는 가장 빠르지만 메모리를 한 번에 많이 사용하므로 파일 크기가 반드시 **작아야 한다**.
- 실제 실행 시간은 버퍼 배열을 직접 다룬 예제와 오차 범위 내로 **거의 비슷하다**.
- 디스크나 파일 시스템이 데이터를 읽고 쓰는 기본 단위가 보통 **4KB 또는 8KB**이기 때문에, 한 번에 모아서 쓴다고 해서 무작정 **더 빠른 것은 아니다**.

## 5. 문자 다루기

- 스트림은 모든 데이터를 `byte` 단위로 처리하므로 `String` 같은 문자를 직접 전달할 수 없다.
- **문자열을 `byte`로 변환하기 (쓰기)**
  - 쓰기 작업을 위해 `String.getBytes(Charset)` 메서드를 활용하여 **`byte` 배열로 변환**한다 (예: `byte[] writeBytes = writeString.getBytes(UTF_8)`).
  - 문자를 숫자로 변경하는 과정이므로 반드시 UTF-8과 같은 **문자 집합(인코딩 셋)** 을 지정해야 한다.
  - 예를 들어 "ABC"를 인코딩하면 `[65, 66, 67]`이라는 `byte` 배열이 된다.
  - 이렇게 만들어진 `byte[]`을 `FileOutputStream`의 `write()`로 전달하면 파일에 문자를 저장할 수 있다.
  - 즉, `String`에 문자 집합을 적용하여 `byte[]`로 인코딩한다.
- **`byte`를 문자열로 복원하기 (읽기)**
  - 파일에서 읽어 들인 데이터를 문자로 복원할 때는 `new String(byte[], Charset)` 생성자를 사용한다 (예: `String readString = new String(readBytes, UTF_8)`).
  - **읽어 온 `byte[]`과 디코딩할 문자 집합을 함께 전달**하면 다시 온전한 문자로 복원할 수 있다.
  - 스트림은 오직 `byte`만 취급하기 때문에 문자를 직접 입출력할 수 없다.
  - 따라서 개발자가 직접 **문자 집합**을 지정하여 변환 과정을 거쳐야 한다.
  - 즉, `byte[]`에 문자 집합을 적용하여 `String`으로 디코딩한다.

### 5.1. OutputStreamWriter, InputStreamReader

#### OutputStreamWriter

- `OutputStreamWriter`는 문자를 입력받고, **받은 문자를 인코딩하여 `byte[]`로 변환**한다.
- 변환한 `byte[]`을 전달할 `OutputStream`과 인코딩 문자 집합 정보가 필요하므로, **생성자를 통해 두 정보를 함께 전달**해야 한다 (예: `new OutputStreamWriter(fos, UTF_8)`).
- `osw.write(writeString)`처럼 **`String` 문자를 직접 전달**할 수 있다.
- 즉, 내부적으로 문자를 인코딩하여 `byte[]`로 변환하고, **그 결과를 `FileOutputStream`에 전달**하여 저장한다.

#### InputStreamReader

- 데이터를 읽을 때는 `read()` 메서드를 사용하여 **문자 하나를 읽어 들인다**.
- 실제 반환 타입은 `int`형이므로, 문자로 다루기 위해서는 **`char`형으로 캐스팅해서 사용**해야 한다 (예: `int ch = read()`).
- 자바의 `char`형은 파일의 끝(EOF)을 의미하는 `-1`을 표현할 수 없기 때문에 **대신 `int`를 반환**하는 것이다.
- 내부적으로는 `FileInputStream`에서 `byte[]`을 읽어오며, 이를 **문자인 `char`로 변경해서 반환**한다.
- `byte`를 문자로 변경하는 과정이므로 반드시 **문자 집합(인코딩 셋)을 지정**해야 한다 (예: `new InputStreamReader(fis, UTF_8)`).

### 5.2. Reader, Writer

- 자바는 데이터를 다루는 방식을 **`byte`** 를 다루는 I/O 클래스와 **문자**를 다루는 I/O 클래스 두 가지로 나누어 제공한다.
- **`byte`를 다루는 I/O 클래스**
  - `OutputStream`과 `InputStream`을 부모 클래스로 상속받아 `byte` 단위의 데이터를 다룬다.
  - 클래스 이름 마지막에 보통 `OutputStream`이나 `InputStream`이 붙어있다.
- **문자를 다루는 I/O 클래스**
  - `Writer`와 `Reader`를 부모 클래스로 상속받아 `String`이나 `char` 같은 문자 데이터를 다룬다.
  - 클래스 이름 마지막에 보통 `Writer`나 `Reader`가 붙어있다.
- **`OutputStreamWriter`의 역할과 원리**
  - 앞에서 다룬 `OutputStreamWriter`는 문자를 다루는 `Writer` 클래스의 자식이므로 `write(String)` 메서드 사용이 가능하다.
  - 문자를 입력받아 `byte`로 변환한 뒤, 이를 다시 `byte`를 다루는 `OutputStream`으로 전달하는 역할을 한다.
  - 여기서 가장 중요한 핵심은 **모든 데이터는 결국 `byte` 단위(숫자)로 저장된다**는 사실이다.
  - 따라서 `Writer`가 문자를 다룬다 해도 그대로 저장할 수는 없으며, 내부적으로 지정된 **문자 집합(Charset)** 을 사용해 문자를 `byte`로 **인코딩**하여 저장한다.

#### FileWriter

- **객체 생성**: `new FileWriter(FILE_NAME, UTF_8)`
  - 파일명과 문자 집합(인코딩 셋)을 전달하여 생성한다.
  - 사실 내부적으로는 스스로 **`FileOutputStream`** 을 하나 생성해서 사용한다.
- **문자로 직접 쓰기**: `fw.write(writeString)`
  - 개발자 입장에서는 문자를 파일에 직접 쓰는 것처럼 편리하게 느껴진다.
  - 하지만 실제로는 `FileWriter` 내부에서 인코딩 셋을 사용해 문자를 `byte`로 변경하고, `FileOutputStream`을 사용해 파일에 저장한다.

#### FileReader

- **작동 방식**: `new FileReader(FILE_NAME, UTF_8)`
  - 앞서 설명한 `FileWriter`와 동일한 원리로 작동한다.
  - 내부에서 **`FileInputStream`** 을 자동으로 생성해서 사용한다.
- **`OutputStreamWriter` / `InputStreamReader`와의 차이점**
  - `FileWriter`는 `OutputStreamWriter`를 상속받은 클래스이며, 다른 특별한 추가 기능은 없다.
  - 유일한 차이점은 기존에 `FileOutputStream`을 직접 생성해 전달하던 번거로움을 줄이고, **생성자 내부에서 자동으로 만들어 준다는 것**이다.
  - 결과적으로 `FileWriter`와 `FileReader`는 기존 클래스를 조금 더 편리하게 사용할 수 있도록 도와주는 **편의성 클래스** 역할을 한다.

### 5.3. BufferedReader, BufferedWriter

```java
FileWriter fw = new FileWriter(FILE_NAME, UTF_8);
BufferedWriter bw = new BufferedWriter(fw, BUFFER_SIZE);
bw.write(writeString);
bw.close();

StringBuilder content = new StringBuilder();
FileReader fr = new FileReader(FILE_NAME, UTF_8);
BufferedReader br = new BufferedReader(fr, BUFFER_SIZE);

String line;
while ((line = br.readLine()) != null) {
    content.append(line).append("\n");
}
br.close();
```

- `BufferedOutputStream`, `BufferedInputStream`처럼 문자 스트림(`Reader`, `Writer`)에도 버퍼를 통한 성능 향상 기능을 제공하는 **`BufferedReader`** 와 **`BufferedWriter`** 클래스가 있다.

#### br.readLine()을 활용한 한 줄 읽기

- 문자를 다룰 때는 보통 **한 줄(라인) 단위**로 처리하는 경우가 많아, `BufferedReader`는 이를 위한 추가 기능을 제공한다.
- `readLine()`은 텍스트를 한 줄씩 읽고 **`String`** 타입으로 반환한다.
- 파일의 끝(**EOF**)에 도달하면 더 이상 읽을 데이터가 없으므로 **`null`** 을 반환한다.
- 반환 타입이 참조형인 `String`이기 때문에 기존 바이트 스트림에서 쓰던 `-1`로 EOF를 표현할 수 없어 대신 `null`을 반환하는 것이다.

## 6. 기타 스트림

### 6.1. PrintStream

```java
FileOutputStream fos = new FileOutputStream("temp/print.txt");
PrintStream printStream = new PrintStream(fos);
```

- 우리가 콘솔 출력을 위해 자주 사용하던 `System.out`의 실제 타입이 바로 **`PrintStream`** 이다.
- `PrintStream`과 `FileOutputStream`을 연결하면, 마치 화면의 콘솔 창에 출력하듯이 데이터를 **파일에 쉽게 출력**할 수 있다.

### 6.2. DataOutputStream / DataInputStream

```java
FileOutputStream fos = new FileOutputStream("temp/data.dat");
DataOutputStream dos = new DataOutputStream(fos);

dos.writeUTF("회원A");
dos.writeInt(20);
dos.writeDouble(10.5);
dos.writeBoolean(true);
dos.close();

FileInputStream fis = new FileInputStream("temp/data.dat");
DataInputStream dis = new DataInputStream(fis);

System.out.println(dis.readUTF());
System.out.println(dis.readInt());
System.out.println(dis.readDouble());
System.out.println(dis.readBoolean());
dis.close();
```

- **기본 데이터 타입 처리**
  - `DataOutputStream`과 `DataInputStream`을 사용하면 자바의 기본 데이터 타입(`String`, `int`, `double`, `boolean` 등)을 번거로운 변환 과정 없이 **그대로 다룰 수 있다**.
  - `FileOutputStream` 등과 조합하면 예제처럼 회원 데이터 같은 자바의 데이터 형을 파일에 아주 **편리하게 저장하고 불러올 수 있다**.
- **사용 시 주의점 (순서 엄수)**
  - 데이터를 읽을(`read`) 때는 반드시 **저장(`write`)한 순서와 타입 그대로 읽어야 한다**. 순서가 어긋나면 바이트가 엉켜 전혀 다른 잘못된 값으로 조회된다.
- **파일이 깨져 보이는 이유**
  - 저장된 `data.dat` 파일을 일반 텍스트 편집기로 열어보면 정상적인 글자로 보이지 않는다.
  - `writeUTF()`는 문자열을 UTF-8 형식으로 저장하지만, 나머지 메서드들은 문자가 아니라 **각 데이터 타입의 실제 `byte` 크기 단위(예: `int`는 4바이트) 그대로 디스크에 기록**하기 때문이다.
  - 텍스트 편집기는 이 순수한 메모리 숫자 데이터(`byte`)들을 억지로 자신의 문자 집합에 맞춰 디코딩하려고 시도하기 때문에, 문자표에 없거나 전혀 예상치 못한 이상한 문자로 깨져서 출력된다.
- **`writeUTF()`와 `readUTF()`의 길이 인식 원리**
  - `writeUTF()`는 UTF-8 형식으로 문자를 저장할 때, 맨 앞에 **2바이트를 추가로 사용하여 글자의 길이를 미리 저장**해둔다 (최대 65535 길이까지 가능).
  - 따라서 `readUTF()`로 읽어 들일 때, 먼저 앞의 2바이트로 길이를 확인하고 딱 그 길이만큼만 문자 데이터를 정확하게 읽어온다.
- **저장 용량 최적화와 단점**
  - 모든 데이터를 문자로 변환해서 저장할 때보다 **저장 용량을 최적화**할 수 있다.
  - 예를 들어 큰 숫자를 문자로 저장하면 자릿수만큼 바이트를 차지하지만, `writeInt()`를 사용하면 숫자의 크기와 상관없이 항상 **4바이트만 사용**하여 효율적이다.
  - 단점은 이렇게 바이트 형태로 직접 저장하면, 일반 문서 편집기로 파일을 열어서 내용을 직접 확인하거나 수정하기가 매우 어렵다는 것이다.
