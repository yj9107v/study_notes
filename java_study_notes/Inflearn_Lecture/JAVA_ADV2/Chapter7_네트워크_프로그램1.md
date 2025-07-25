# 📚 Chapter 7: 네트워크 - 프로그램 1

> 📌 공부 날짜: 2025/05/24
> - `JAVA `- 고급 2편
> - `References`: 인프런 - 김영한의 실전 자바

---

## ✅ 네트워크 프로그램 1 - 예제

- 여기서는 TCP/IP로 개발할 예정이다.
    - (UDP는 직접 사용할 일이 많지 않으므로 다루지 않음.)

```java
public class ClientV1 {
    private static final int PORT = 12345;

    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("localhost", PORT);
        DataInputStream input = new DataInputStream(socket.getInputStream());
        DataOutputStream output = new DataOutputStream(socket.getOutputStream());

        String toSend = "Hello"; // 서버에게 문자 보내기
        output.writeUTF(toSend);

        String received = input.readUTF(); // 서버로부터 문자 받기

        // 자원 정리
        input.close();
        output.close();
        socket.close();
    }
}
```

```java
public class Server1 {
    private static final int PORT = 12345;

    public static void main(String[] args) {
        ServerSocket serverSocket = new ServerSocket(PORT);

        Socket socket = serverSocket.accept();
        DataInputStream input = new DataInputStream(socket.getInputStream());
        DataOutputStream output = new DataOutputStream(socket.getOutputStream());
        
        String received = input.readUTF(); // 클라이언트로부터 문자 받기
        
        String toSend = received + "World!"; // 클라이언트에게 문자 보내기
        output.writeUTF(toSend);
        
        // 자원 정리
        input.close();
        output.close();
        socket.close();
        serverSocket.close();
    }
}
```
- **서버를 먼저 실행하고, 그다음에 클라이언트를 실행해야 한다.**
- `localhost`는 현재 사용 중인 컴퓨터 자체를 가리키는 특별한 호스트 이름이다.
  - `google.com`, `naver.com`과 같은 호스트 이름이지만, 자기 자신의 컴퓨터를 뜻하는 이름이다.
- `localhost`는 127.0.0.1이라는 IP로 매핑된다.
- 127.0.0.1은 IP 주소 체계에서 루프백 주소(loopBack address)로 지정된 특별한 IP 주소이다.
  - 이 주소는 컴퓨터가 스스로를 가리킬 때 사용되며, "localhost"와 동일하게 취급된다.
- 127.0.0.1은 컴퓨터가 네트워크 인터페이스를 통해 외부로 나가지 않고, 자신에게 직접 네트워크 패킷을 보낼 수 있도록 한다.

> ⚠️ **주의 - localhost**
> - 만약 `localhost`가 잘 되지 않는다면, 자신의 PC의 운영체제에 `localhost`가 127.0.0.1로 매핑되어 있지 않은 문제이다.
> - 이 경우 `localhost` 대신에 127.0.0.1을 직접 입력하면 된다.

- **주의❗ - 서버 연결 불가**
```text
java.net.ConnectException: Connection refused 
```
- 서버를 시작하지 않고, 클라이언트만 실행하면 해당 예외가 발생한다.

> ⚠️ **주의 - BindException**
> ```text
> Exception in thread "main" java.net.BindException: Address already in use
> ```
> - 만약 이런 예외가 발생했다면, 이미 12345라는 포트를 다른 프로세스가 사용되고 있다는 뜻이다.
> - 인텔리J를 종료할 때 반드시 `Terminate`로 종료해야 한다.
>   - `Disconnect`로 종료하면 12345 포트를 점유하고 있는 서버 프로세스가 그대로 살아있는 상태로 유지된다.
>   - 따라서 다시 실행할 경우 12345 포트를 사용하지 못한다.

---

## ✅ 네트워크 프로그램 1 - 분석
- TCP/IP 통신에서는 통신할 대상 서버를 찾을 때 호스트 이름이 아니라, IP 주소가 필요하다.
- 네트워크 프로그램을 분석하기 전에 먼저 호스트 이름으로 IP를 어떻게 찾는지 확인해 보자.

### 📚 DNS 탐색
```java
public class InetAddressMain { 
    public static void main(String[] args) {
        InetAddress localhost = InetAddress.getByName("localhost");
    
        InetAddress google = InetAddress.getByName("google.com");
    }
}
```
- 자바의 `InetAddress` 클래스를 사용하면 호스트 이름으로 대상 IP를 찾을 수 있다.
    1. 자바는 `InetAddress.getByName(호스트명)` 메서드를 사용해서 해당하는 IP 주소를 조회한다.
    2. 이 과정에서 시스템의 호스트 파일을 먼저 확인한다.
        - `/etc/hosts` (리눅스, mac)
        - `C:\Windows\System32\drivers\etc\hosts` (윈도우, Windows)
    3. 호스트 파일에 정의되어 있지 않다면, DNS 서버에 요청해서 IP 주소를 얻는다.

---

### 📚 클라이언트 코드 분석
#### 🔍 클라이언트와 서버의 연결의 `Socket`을 사용한다.
```java
Socket socket = new Socket("localhost", PORT);
```
- `localhost`를 통해 자신의 컴퓨터에 있는 12345 포트에 TCP 접속을 시도한다.
  - `localhost`는 IP가 아니므로 해당하는 IP를 먼저 찾는다. 내부에서 `InetAddress`를 사용.
  - `localhost`는 127.0.0.1이라는 IP에 매핑되어 있어, `127.0.0.1:12345`에 TCP 접속을 시도한다.
- 연결이 성공적으로 완료되면 `Socket` 객체를 반환한다.
- `Socket`은 서버와 연결되어 있는 **연결점**이라고 생각하면 된다.
- `Socket` 객체를 통해서 서버와 통신할 수 있다.

#### 🔍 클라이언트와 서버 간의 데이터 통신은 Socket이 제공하는 스트림을 사용한다.
```java
DataInputStream input = new DataInputStream(socket.getInputStream());
DataOutputStream output = new DataOutputStream(socket.getOutputStream());
```
- `Socket`은 서버와 데이터를 주고받기 위한 스트림을 제공한다.
- `InputStream`: 서버에서 전달한 데이터를 클라이언트가 받을 때 사용.
- `OutputStream`: 클라이언트에서 서버에 데이터를 전달할 때 사용한다.
- `InputStream`, `OutputStream`을 그대로 사용하면 모든 데이터를 `byte`로 변환해서 전달해야 하기 때문에 번거롭다.
  - 여기서는 `DataInputStream`, `DataOutputStream`이라는 보조 스트림을 사용해서, 자바 타입의 메시지를 편리하게 주고받을 수 있도록 했다.
    - 다른 보조 스트림도 가능.


- **❗ 사용한 자원은 반드시 정리해야 한다.**

---

### 📚 서버 코드 분석
#### 🔍 서버 소켓
- 서버는 특정 포트를 열어두어야 한다. 그래야 클라이언트가 해당 포트를 지정해서 접속할 수 있다.
```java
ServerSocket serverSocket = new ServerSocket(PORT); 
```
- 서버는 서버 소켓(`ServerSocket`)이라는 특별한 소켓을 사용한다.
- 지정한 포트를 사용해서 서버 소켓을 생성하면, 클라이언트는 해당 포트로 서버에 연결할 수 있다.

> 🔍 클라이언트와 서버의 연결 과정
> - 서버가 12345 포트로 서버 소켓을 열어둔다. 클라이언트는 이제 12345 포트로 서버에 접속할 수 있다.
> - 클라이언트가 12345 포트에 연결을 시도.
> - 이때 OS 계층에서 TCP 3 way handshake가 발생하고, TCP 연결이 완료된다.
> - TCP 연결이 완료되면 서버는 OS backlog queue라는 곳에 클라이언트와 서버의 TCP 연결 정보를 보관한다.
>   - 이 연결 정보를 보면 클라이언트의 IP, PORT, 서버의 IP, PORT 정보가 모두 들어있다.

#### 🔍 클라이언트와 랜덤 포트
- TCP 연결 시에는 클라이언트 서버 모두 IP, 포트 정보가 필요하다.
- **클라이언트**: localhost(127.0.0.1), 50000(포트 랜덤 생성)
- **서버**: localhost, 12345
- **서버의 경우 포트가 명확하게 지정되어 있어야 한다.**
    - 그래야 클라이언트에서 서버에 어떤 포트에 접속할지 알 수 있다.
    - 반면에 서버에 접속하는 클라이언트의 경우에는 자신의 포트가 명확하게 지정되어 있지 않아도 된다.
    - 클라이언트는 보통 포트를 생략하는데, 생략할 경우 클라이언트 PC에 남아있는 포트 중 하나가 랜덤으로 할당된다.
    - 참고로 클라이언트의 포트도 명시적으로 할당할 수는 있지만 잘 사용❌.

#### 🔍 accept()
```java
Socket socket = serverSocket.accept();
```
- **서버 소켓은 단지 클라이언트와 서버의 TCP 연결만 지원하는 특별한 소켓이다.**
- 실제 클라이언트와 서버가 정보를 주고받으려면 `Socket` 객체가 필요함. (**서버 소켓이 아니다❗ 소켓이다❗**)
- `serverSocket.accept()` 메서드를 호출하면 TCP 연결 정보를 기반으로, `Socket` 객체를 만들어서 반환한다.

> `accpet()` 호출 과정
> - `accept()`를 호출하면 backlog queue에서 TCP 연결 정보를 조회한다.
>   - 만약 TCP 연결 정보가 없다면, 연결 정보가 생성될 때까지 대기한다. (블로킹)
> - 해당 정보를 기반으로 `Socket` 객체를 생성한다.
> - 사용한 TCP 연결 정보는 backlog queue에서 제거된다.

- **자바가 해주는 건 정보를 가져와 보내는 것. 나머진 OS가 처리(연결, 저장 등)**

---

### 🤔 문제
- 이 프로그램은 메시지를 하나만 주고받으면 클라이언트와 서버가 모두 종료된다. 메시지를 계속 주고받고, 원할 때 종료할 수 있도록 변경해야 한다.

---

## ✅ 네트워크 프로그램 2 - 예제
- 클라이언트와 서버가 메시지를 주고받는 부분만 `while`로 반복하면 된다.

```java
// 클라이언트 코드
while (true) {
    String toSend = scanner.nextLine();
    output.writeUTF(toSend); // 서버: received = input.readUTF();
    
    if (toSend.equals("exit")) break; // 서버: received.equals("exit")
    
    String received = input.readUTF(); // 서버: String toSend = received + " World!"
                                       //      output.writeUTF(toSend);
}
```

### 🤔 문제
- 서버는 하나의 클라이언트가 아니라, 여러 클라이언트의 연결을 처리할 수 있어야 한다.
- 새로운 클라이언트가 접속하면 현재 코드는 정상 수행이 되지 않는다.

---

## ✅ 네트워크 프로그램 2 - 분석
### 📚 서버 소켓과 연결 더 자세히
- 이번에는 여러 클라이언트가 서바에 접속한다고 가정.
- 50000번 클라이언트와 60000번 클라이언트 모두 서버와 연결하였고, 클라이언트의 소켓도 정상 생성된다.
- 서버가 클라이언트와 데이터를 주고받으려면 소켓을 획득해야 한다.
- `ServerSocket.accept()` 메서드를 호출하면 backlog 큐의 정보를 기반으로 소켓 객체를 하나 생성한다.
- 큐이므로 순서대로 데이터를 꺼낸다. 50000번 클라이언트의 접속 정보를 기반으로 서버에 소켓이 하나 생성된다.
- 60000번 클라이언트도 서버와 TCP 연결이 되었기 때문에 서버로 메시지를 보낼 수 있다.
  - 아직 서버에 Socket 객체가 없더라도 메시지는 보낼 수 있다. TCP 연결은 이미 완료되었다.

> 데이터를 주고받는 과정
> - **클라이언트가 "Hello, world!"라는 메시지를 서버에 전송하는 경우**
>   - 클라이언트
>     - 애플리케이션 ➡️ **OS TCP 송신 버퍼 ➡️ 클라이언트 네트워크 카드**
>   - 클라이언트가 보낸 메시지가 서버에 도착했을 때, 서버
>     - 서버 네트워크 카드 ➡️ **OS TCP 수신 버퍼** ➡️ 애플리케이션

- 여기서 60000번 클라이언트가 보낸 메시지는 서버 애플리케이션에서 아직 읽지 않았기 때문에, **서버 OS의 TCP 수신 버퍼에서 대기하게 된다.**


- **여기서 핵심적인 내용이 있는데, 소켓 객체 없이 서버 소켓만으로 TCP 연결은 완료된다는 점이다.**
  - (서버 소켓은 연결만 담당)
- **하지만 연결 이후에 서로 메시지를 주고받으려면 소켓 객체가 필요하다.**

---

### 📚 예제 2 코드의 문제
- 둘 이상의 클라이언트가 작동하지 않는 이유❗
  - 새로운 클라이언트가 접속하면?
    - 서버의 main 스레드는 `accept()` 메서드를 절대로 호출할 수 없다.
    - 이유는 `while`문으로 기존 클라이언트와 메시지를 주고받는 부분만 반복하기 때문이다.
    - `accept`를 호출해야 소켓 객체를 생성하고 클라이언트와 메시지를 주고받을 수 있다.
  - 2개의 블로킹 작업 - 핵심은 별도의 스레드가 필요하다❗
    - `accept()`: 클라이언트와 서버의 연결을 처리하기 위해 대기
    - `readXxx()`: 클라이언트의 메시지를 받아서 처리하기 위해 대기
    - 각각의 블로킹 작업은 별도의 스레드에서 처리해야 한다. 그렇지 않으면 다른 블로킹 메서드 때문에 계속 대기할 수 있다.

---

## ✅ 네트워크 프로그램 3
- 이번에는 여러 클라이언트가 동시에 접속할 수 있는 서버 프로그램을 작성해 보자.


- 서버의 `main` 스레드는 서버 소켓을 생성하고, 서버 소켓의 `accept()`를 반복해서 호출해야 한다.
- 클라이언트가 서버에 접속하면 서버 소켓의 `accpet()` 메서드가 `Socket`을 반환한다.
- `main` 스레드는 이 정보를 기반으로 `Runnable`을 구현한 `Session`이라는 별도의 객체를 만들고, 새로운 스레드에서 이 객체를 실행한다.
  - 여기서는 `Thread-0`이 `Session`을 실행한다.
- `Session` 객체와 `Thread-0`은 연결된 클라이언트와 메시지를 주고받는다.
- 새로운 TCP 연결이 발생하면 `main` 스레드는 새로운 `Session` 객체를 별도의 스레드에서 실행한다. 그리고 이 과정을 반복한다.

#### 🔍 역할의 분리
- `main` 스레드
  - `main` 스레드는 새로운 연결이 있을 때마다 `Session` 객체와 별도의 스레드를 생성하고, 별도의 스레드가 `Session` 객체를 실행하도록 한다.
- `Session` 담당 스레드
  - `Session`을 담당하는 스레드는 자신의 소켓이 연결된 클라이언트와 메시지를 반복해서 주고받는 역할을 담당.

```java
public class SessionV3 implements Runnable {
    private final Socket socket;
    
    public SessionV3(Socket socket) {
        this.socket = socket;
    }
    
    @Override
    public void run() {
        try {
            DataInputStream input = new DataInputStream(socket.getInputStream());
            DataOutputStream output = new DataOutputStream(socket.getOutputStream());
            
            while (true) {
                String received = input.readUTF(); // 블로킹
              
                if (received.equals("exit")) break;
                
                String toSend = received + " World!";
                output.writeUTF(toSend);
            }
            
            // 자원 정리
            input.close();
            output.close();
            socket.close();
        } catch (IOException e) {
            throw new RunTimeException(e);
        }
}
```
- `Runnable`을 구현해서 별도의 스레드에서 실행한다.

```java
public class ServerV3 {
    private static final int PORT = 12345;

  public static void main(String[] args) throws IOException {
      ServerSocket serverSocket = new ServerSocket(PORT);
      
      while (true) {
        Socket socket = serverSocket.accept(); // 블로킹
        
        SessionV3 session = new SessionV3(socket);
        Thread thread = new Thread(session);
        thread.start(); // session의 run() 호출
      }
  }
}
```
- `main` 스레드는 서버 소켓을 생성하고, `serverSocket.accept()`을 호출해서 연결을 대기한다.
- 새로운 연결이 추가될 때마다 `Session` 객체를 생성하고 별도의 스레드에서 `Session` 객체를 실행한다.
- 이 과정을 반복.


- **블로킹되는 부분은 이렇게 별도의 스레드로 나누어 실행해야 한다.**

#### 🤔 문제
- **여기서 실행 중인 클라이언트를 인텔리J의 빨간색 Stop 버튼을 눌러서 직접 종료할 시?**
- 클라이언트의 연결을 직접 종료하면 클라이언트 프로세스가 종료되면서, 클라이언트와 서버의 TCP 연결도 함께 종료된다.
  - 이때 서버에서 `readUTF()`로 클라이언트가 메시지를 읽으려고 하면 `EOFException`이 발생.
  - 소켓의 TCP 연결이 종료되었기 때문에 더는 읽을 수 있는 메시지가 없다는 뜻이다.
  - EOF(파일의 끝)가 여기서는 전송의 끝이라는 뜻이다.
- **그런데 여기서 심각한 문제가 하나 있다. 이렇게 예외가 발생해 버리면 서버에서 자원 정리 코드를 호출하지 못한다는 점이다.**

#### 🔍 윈도우 OS 내용 추가
- **클라이언트를 직접 종료한 경우 서버 로그 - 윈도우**
- 참고로 윈도우의 경우 `java.io.EOFException`이 아니라 `java.net.SocketException: Connection reset`이 발생한다.
- 클라이언트의 연결을 직접 종료하면 클라이언트 프로세스가 종료되면서, 클라이언트와 서버의 TCP 연결도 함께 종료된다.
  - 이때 소켓을 정상적으로 닫지 않고 프로그램을 종료했기 때문에 각각의 OS는 남아있는 TCP 연결을 정리하려고 시도한다.
  - 이때 MAC과 윈도우 OS의 TCP 연결 정리 방식이 다르다.
    - MAC: TCP 연결 정상 종료
    - 윈도우: TCP 연결 강제 종료


- 자바 객체는 GC가 되지만, 자바 외부의 자원은 자동으로 GC가 되는 게 아니다. 따라서 꼭! 정리를 해주어야 한다.
- (TCP 연결의 경우 운영체제가 어느 정도 연결을 정리해 주지만, 직접 연결을 종료할 때보다 더 많은 시간이 걸릴 수 있다.)


#### ❗ 자원 정리를 제대로 해주어야 한다.

---

## ✅ 자원 정리
### 📚 자원 정리 1
- `call()`: 정상 로직 호출
- `callEx()`: 비정상 로직 호출 `CallException`을 던진다.
- `close()`: 정상 종료
- `closeEx()`: 비정상 종료, `CloseException`을 던진다.

```java
public class ResourceCloseMainV1 {
  public static void main(String[] args) {
      try {
          logic();
      } catch (CallException e) {
          e.printStackTrace();
      } catch (CloseException e) {
          e.printStackTrace();
      }
  }
  
  private static void logic() throws CallException, CloseException {
      ResourceV1 resource1 = new ResourceV1("resource1");
      ResourceV1 resource2 = new ResourceV2("resource2");
      
      resource1.call();
      resource2.callEx(); // CallException
    
      // 호출 안 됨
      resource2.closeEx();
      resource1.closeEx();
  }
}
```
- 서로 관련된 자원은 나중에 생성한 자원을 먼저 정리해야 한다.
- 예를 들어서 `resouce1`을 생성하고, `resource1`의 정보를 활용해서 `resource2`를 생성한다면, 닫을 때는 그 반대인 `resource2`를 먼저 닫고, 그다음에 `resource1`을 닫아야 한다.
  - 왜냐하면 `resource2`의 입장에서 `resource1`의 정보를 아직 참고하고 있기 때문이다.

#### 🤔 문제
- `callEx()`를 호출하면서 예외가 발생했다.
  - 예외 때문에 자원 정리 코드가 정상 호출되지 않았다.
- 이 코드는 예외가 발생하면 자원이 정리되지 않는다는 문제가 있다.

---

### 📚 자원 정리 2
- 이번에는 예외가 발생해도 자원을 정리하도록 해본다.

```java
private static void logic() throws CallException, CloseException {
    ResourceV1 resource1 = null;
    ResourceV1 resource2 = null;
    
    try {
        resource1 = new ResourceV1("resource1");
        resource1 = new ResourceV2("resource2");
        
        resource1.call();
        resource2.callEx();
    } catch (CallException e) {
        throw e; // CallException 다시 던짐
    } finally {
        if (resource2 != null) resource2.closeEx(); // CloseException 발생!
        if (resource1 != null) resource1.closeEx(); // 이 코드 호출 안 됨!
    }
}
```
#### 🔍 null 체크
- 이번에는 `finally` 코드 블록을 사용해서 자원을 닫는 코드가 항상 호출되도록 했다.
- 만약 `resource2` 객체를 생성하기 전에 예외가 발생하면 `resource2`는 `null`이 된다.
  - 따라서 `null` 체크를 해야 한다.
- `resource1`의 경우에도 `resource1`을 생성하는 중에 예외가 발생하면 `null` 체크가 필요하다.

#### 🔍 자원 정리 중에 예외가 발생하는 문제
- `finally` 코드 블록은 항상 호출되기 때문에 자원이 잘 정리될 것 같지만, 이번에는 자원을 정리하는 중에 `finally` 코드 블록 안에서 `resource2.closeEx()`를 호출하면서 예외가 발생한다.
- 결과적으로 `resource1.closeEx`는 호출되지 않는다.

#### ⚠️ 핵심 예외가 바뀌는 문제
- 이 코드에서 발생한 핵심적인 예외는 `CallException`이다. 이 예외 때문에 문제가 된 것이다.
  - 그런데 `finally` 코드 블록에서 자원을 정리하면서 `CloseException` 예외가 추가로 발생했다.
  - 예외 때문에 자원을 정리하고 있는데, 자원 정리 중에 또 예외가 발생한 것이다.
  - 이 경우 `logic()`을 호출한 쪽에서는 핵심 예외인 `CallException`이 아니라 `finally` 블록에서 새로 생성된 `CloseException`을 받게 된다.
    - 핵심 예외가 사라진 것이다!
- 개발자가 원하는 예외는 당연히 핵심 예외다.
  - 이 핵심 예외를 확인해야 제대로 된 문제를 찾을 수 있다.
  - 자원을 닫는 중에 발생한 예외는 부가 예외일 뿐이다.

#### 🤔 문제
- `close()` 시점에 실수로 예외를 던지면, 이후 다른 자원을 닫을 수 없는 문제 발생
- `finally()` 블록 안에서 자원을 닫을 때 예외가 발생하면, 핵심 예외가 `finally`에서 발생한 부가 예외로 바뀌어 버린다.
  - 그리고 핵심 예외가 사라진다.

---

### 📚 자원 정리 3
- 자원 정리의 코드에서 try-catch를 사용해서 자원 정리 중에 발생하는 예외를 잡아보자.

```java
finally {
    if (resource2 != null) {
        try {
            resource2.closeEx();
        } catch (CloseException e) {
            // close()에서 발생한 예외는 버린다. 필요하면 로깅 정도
            System.out.println("close ex: " + e);
        }
    }
    if (resource1 != null) {
        try {
            resource1.closeEx();
        } catch (CloseException e) {
            System.out.println("close ex: " + e);
        }
    }
}
```
- `finally` 블록에서 각각의 자원을 닫을 때도, 예외가 발생하면 예외를 잡아서 처리하도록 했다.
- 자원 정리 시점에 발생한 예외를 잡아서 처리했기 때문에, 자원 정리 시점에 발생한 부가 예외가 핵심 예외를 가리지 않는다.
- 자원 정리 시점에 발생한 예외는 당장 더 처리할 수 있는 부분이 없다.
  - 이 경우 로그를 남겨서 개발자가 인지할 수 있게 하는 정도면 충분하다.

#### 🤔 문제
- 이전에 발생했던 2가지 문제를 해결했다.
- 핵심적인 문제들은 해결되었지만 코드 부분에서 보면 아쉬운 부분이 많다.
  - `resource` 변수를 선언하면서 동시에 할당할 수 없음(`try`, `finally` 코드 블록과 변수 스코프가 다른 문제)
  - `catch` 이후에 `finally` 호출, 자원 정리가 조금 늦어진다.
  - 개발자가 실수로 `close()`를 호출하지 않을 가능성
  - 개발자가 `close()` 호출 순서를 실수, 보통 자원을 생성한 순서와 반대로 닫아야 함.

- 지금까지 수많은 자바 개발자들이 자원 정리 때문에 고통받아 왔다.
- **이런 문제를 한 번에 해결하는 것이 바로 `try-with-resources` 구문이다.**

---

### 📚 자원 정리 4
```java
// try-with-resources를 써서 자동 자원 반납을 하려면 AutoCloseable을 구현해 줘야 한다.
public class ResourceV2 implements AutoCloseable {
    @Override
    public void close() throws CloseException {
        System.out.println(name + " close");
        throw new CloseException(name + " ex");
    }
}
```
- `AutoCloseable`을 구현했다.
- `close()`는 항상 `CloseException`을 던지도록 했다.

```java
public class ResourceCloseMainV4 {
  public static void main(String[] args) {
    try {
        logic();
    } catch (CallException e) {
        Throwable[] suppressed = e.getSuppressed();
        for (Throwable throwable : suppressed) {
            System.out.println("suppressedEx = " + throwable);
        }
        e.printStackTrace();
    } catch (CloseException e) {
        e.printStackTrace();
    }
  }
  
  private static void logic() throws CallException, CloseException {
      try (ResourceV2 resource1 = new ResourceV2("resource1");
           ResourceV2 resource2 = new ResourceV2("resource2")) {
          
          resource1.call();
          resource2.callEx(); // CallException;
      } catch (CallException e) {
          System.out.println("ex: " + e);
          throw e;
      }
  }
}
```
- `try-with-resources`는 단순하게 `close()`를 자동 호출해 준다는 정도의 기능만 제공하는 것이 아니다.
- 고민한 6가지 문제를 모두 해결하는 장치이다.

#### 🤔 2가지 핵심 문제
- `close()` 시점에 실수로 예외를 던지면, 이후에 다른 자원을 닫을 수 없는 문제 발생
- `finally()` 블록 안에서 자원을 닫을 때 예외가 발생하면, 핵심 예외가 `finally`에서 발생한 부가 예외로 바뀌어 버린다.
  - 그리고 핵심 예외가 사라진다.

#### 🤔 4가지 부가 문제
- `resource` 변수를 선언하면서 동시에 할당할 수 없음(`try`, `finally` 코드 블록과 변수 스코프가 다른 문제)
- `catch` 이후에 `finally` 호출, 자원 정리가 조금 늦어진다.
- 개발자가 실수로 `close()`를 호출하지 않을 가능성
- 개발자가 `close()` 호출 순서를 실수, 보통 자원을 생성한 순서와 반대로 닫아야 함.

#### 🔍 Try with resources 장점
1. **리소스 누수 방지**
  - 모든 리소스가 제대로 닫히도록 보장한다.
    - 실수로 `finally` 블록을 적지 않거나, `finally` 블록 안에서 자원 해제 코드를 누락하는 문제들을 예방할 수 있다.

2. **코드 간결성 및 가독성 향상**
  - 명시적인 `close()` 호출이 필요 없어 코드가 더 간결하고 읽기 쉬워진다.

3. **스코프 범위 한정**
  - 예를 들어 리소스로 사용되는 `resource1, 2` 변수의 스코프가 `try` 블록 안으로 한정된다.
    - 따라서 코드 유지보수가 더 쉬워진다.

4. **조금 더 빠른 자원 해제**
  - 기존에는 try -> catch -> finally로 catch 이후에 자원을 반납했다.
    - Try with resources 구분은 `try` 블록이 끝나면 즉시 `close()`를 호출한다.

5. **자원 정리 순서**
  - 먼저 선언한 자원을 나중에 정리한다.

6. **부가 예외 포함**

#### 🔍 Try with resources 예외 처리와 부가 예외 포함
- `try-with-resources`를 사용하는 중에 핵심 로직 예외와 자원을 정리하는 중에 발생하는 부가 예외가 모두 발생하면 어떻게 될까?
  - `try-with-resources`는 핵심 예외를 반환한다.
  - 부가 예외는 핵심 예외 안에 `Suppressed`로 담아서 반환한다.
  - 개발자는 자원 정리 중에 발생한 부가 예외를 `e.getSuppressed()`를 통해 활용할 수 있다.
- **`try-with-resources`를 사용하면 핵심 예외를 반환하면서, 동시에 부가 예외도 필요하면 확인할 수 있다.**


- 참고로 자바 예외에는 `e.addSuppressed(ex)`라는 메서드가 있어서 예외 안에 참고할 예외를 담아둘 수 있다.

---