# 📚 Chapter 8: 네트워크 - 프로그램 2

> 📌 공부 날짜: 2025/05/26
> - `JAVA `- 고급 2편
> - `References`: 인프런 - 김영한의 실전 자바

---

## ✅ 네트워크 프로그램 4, 5 - 자원 정리 1, 2
- `try-with-resources`를 항상 사용할 수 있는 것은 아니고, `finally`에서 직접 자원을 정리해야 하는 경우가 많이 있다.
- 참고로 `OutputStream`, `InputSteram`, `Socket` 모두 `AutoCloseable`을 구현하고 있어, `try-with-resources`를 사용할 수 있다.

```java
public class SessionV5 implements Runnable {
    private final Socket socket;
    
    public SessionV5 (Socket socket) {
        this.socket = socket;
    }
    
    @Override
    public void run() {
        try (socket;
             DataInputStream input = new DataInputStream(socket.getInputStream());
             DataOutputStream output = new DataOutputStream(socket.getOutputStream())) 
        {   // 클라이언트 문자 받기, 보내기 등 내용
        }
    }
}
```
- `Socket` 객체의 경우 `Session`에서 직접 생성하는 것이 아니라 외부에서 받아오는 객체이다.
  - 이 경우 `try` 선언부에 예제와 같이 객체의 참조를 넣어두면 자원 정리 시점에 `AutoCloseable`이 호출된다.

---

## ✅ 네트워크 프로그램 6 - 자원 정리 3
- 서버를 종료할 때, 서버 소켓과 연결된 모든 소켓 자원을 다 반납하고 서버를 안정적으로 종료하는 방법을 알아보자.
- 서버를 종료하려면 서버에 종료라는 신호를 전달해야 한다.

#### 🔍 셧다운 훅(Shutdown Hook)
- 자바는 프로세스가 종료될 때, 자원 정리나 로그 기록과 같은 종료 작업을 마무리할 수 있는 셧다운 훅이라는 기능을 지원한다.
- 프로세스 종료는 크게 2가지로 분류할 수 있다.
  - **정상 종료**
    - 모든 non 데몬 스레드의 실행 완료로 자바 프로세스 정상 종료
    - 사용자가 Ctrl+C를 눌러서 프로그램을 중단
    - `kill` 명령 전달 (`kill -9` 제외)
    - IntelliJ의 stop 버튼
  - **강제 종료**
    - 운영체제에서 프로세스를 더 이상 유지할 수 없다고 판단할 때 사용
    - 리눅스/유닉스의 `kill -9`나 Windows의 `taskkill /F`


- 정상 종료의 경우에는 셧다운 훅이 작동해서 프로세스 종료 전에 필요한 후 처리를 할 수 있다.
- 반면에 강제 종료의 경우에는 셧다운 훅이 작동하지 않는다.


- 클라이언트 코드는 `try-with-resources`로 자원 정리 가능.
- 서버는 세션을 관리하는 세션 매니저가 필요하다.
```java
// 동시성 처리
public class SessionManagerV6 {
    private List<SessionV6> sessions = new ArrayList<>();
    
    public synchronized add(SessionV6 session) {
        sessions.add(session);        
    }
    
    public synchronized remove(SessionV6 session) {
        sessions.remove(session);
    }
    
    public synchronized closeAll() {
      for (SessionV6 session : sessions) {
          session.close();
      } 
      sessions.clear();
    }
}
```
- 각 세션은 소켓과 연결 스트림을 가지고 있다. 따라서 서버를 종료할 때 사용하는 세션들도 함께 종료해야 한다.
  - 모든 세션들을 찾아서 죵료하려면 생성한 세션을 보관하고 관리할 객체가 필요하다.

#### 🔍 SessionManager
- `add()`: 클라이언트와 새로운 연결을 통해, 세션이 새로 만들어지는 경우 `add()`를 호출해서 세션 매니저에 세션을 추가한다.
- `remove()`: 클라이언트의 연결이 끊어지면 세션도 함께 정리된다. 이 경우 `remove()`를 호출해서 세션 매니저에서 세션을 제거한다.
- `closeAll()`: 서버를 종료할 때 사용하는 세션들도 모두 닫고, 정리한다.

```java
public class SessionV6 implements Runnable {
    private final Socket socket;
    private final DataInputStream input;
    private final DataOutputStream output;
    private final SessionManagerV6 sessionManager;
    private boolean closed = false;
    
    public SessionV6 (Socket socket, SessionManagerV6 sessionManger) throws IOException {
        this.socket = socket;
        input = new DataInputStream(socket.getInputStream());
        output = new DataOutputStream(socket.getOutputStream());
        this.sessionManager = sessionManger;
        this.sessionManager.add(this);
    }
    
    @Override
    public void run() {
        try {
          while (true) {
            String received = input.readUTF();
            
            if (received.equals("exit")) break;
            
            String toSend = received + "World!";
            output.writeUTF(toSend);
          }
        } catch (IOException e) {
            log(e);
        } finally {
            sessionManager.remove(this);
            close();
        }
    }
    
    public synchronized void close() {
        if (closed) return;
        
        closeAll(socket, input, output); // SocketCloseUtil 클래스를 만들어 자원 정리함 -> null 여부 확인 및 try, catch
        closed = true;
    }
}
```
- `Session`은 이제 `try-with-resources`를 사용할 수 없다.
- 왜냐하면 서버를 종료하는 시점에도 `Session`의 자원을 정리해야 하기 때문이다.

> **try-with-resources는 사용과 해제를 함께 묶어서 처리할 때 사용한다.**
> - `try-with-resources`는 try 선언부에서 사용한 자원을 try가 끝나는 시점에 정리한다.
> - 따라서 try에서 자원의 선언과 자원 정리를 묶어서 처리할 때 사용할 수 있다.
> - 하지만 지금은 서버를 종료하는 시점에도 `Session`이 사용하는 자원을 정리해야 한다.
> - 서버를 종료하는 시점에 자원을 정리하는 것은 `Session` 안에 있는 `try-with-resources`를 통해 처리할 수 없다.

#### 🔍 동시성 문제
- 자원을 정리하는 `close()` 메서드는 2곳에서 호출될 수 있다.
  - 클라이언트와 연결이 종료되었을 때(`exit` 또는 예외 발생)
  - 서버를 종료할 때
- 따라서 `close()`가 다른 스레드에서 동시에 중복 호출될 가능성이 있다.
- 이런 문제를 막기 위해 `synchronized` 키워드를 사용했다.
  - 그리고 자원 정리 코드가 중복 호출되는 것을 막기 위해 `closed` 변수를 플래그로 사용했다.

---

## ✅ 네트워크 프로그램 6 - 자원 정리 4
```java
public class ServerV6 {
    private static final int port = 12345;

  public static void main(String[] args) throws IOException {
    SessionManagerV6 sessionManager = new SessionManagerV6();
    ServerSocket serverSocket = new ServerSocket(PORT);
    
    // ShutdownHook 등록
    ShutdownHook shutdownHook = new ShutdownHook(serverSocket, sessionManager);
    Runtime.getRuntime().addShutdownHook(new Thread(shutdownHook, "shutdown"));
    
    try {
      while (true) {
        Socket socket = serverSocket.accpet();
        
        SessionV6 session = new SessionV6(socket, sessionManager);
        Thread thread = new Thread(session);
        thread.start();
      }
    } catch (IOException e) {
        log(e);
    }
  }
  
  static class ShutdownHook implements Runnable {
      private final ServerSocket serverSocket;
      private final SessionManagerV6 sessionManager;

    public ShutdownHook(ServerSocket serverSocket, SessionManagerV6 sessionManager) {
        this.serverSocket = serverSocket;
        this.sessionManager = sessionManager;
    }
    
    @Override // 셧다운 훅 실행 코드
    public void run() {
        try {
            sessionManager.closeAll();
            serverSocket.close();
            
            Thread.sleep(1000); // 자원 정리 대기
        } catch (Exception e) {
            e.printStackTrace();
          System.out.println("e = " + e);
        }
    }
  }
}
```

#### 🔍 셧다운 훅 등록
- `Runtime.getRuntime().addShutdownHook()`을 사용하면 자바 종료 시 호출되는 셧다운 훅을 등록할 수 있다.
- 여기에 셧다운이 발생했을 때 처리할 작업과 스레드를 등록하면 된다.

#### 🔍 셧다운 훅 실행 코드
- 셧다운 훅이 실행될 때 모든 자원을 정리한다.
- `sessionManager.closeAll()`: 모든 세션이 사용하는 자원(`Socket`, `InputStream`, `OutputStream`)을 정리한다.
- `serverSocket.close()`: 서버 소켓을 닫는다.

#### 🔍 자원 정리 대기 이유
```java
Thread.sleep(1000);
```
- 보통 모든 non 데몬 스레드의 실행이 완료되면 자바 프로세스가 정상 종료된다. 하지만 다음과 같은 종료도 있다.
  - 사용자가 Ctrl+C를 눌러서 프로그램을 중단
  - `kill` 명령 전달(`kill -9` 제외)
  - IntelliJ의 stop 버튼
- 이런 경우에는 non 데몬 스레드의 종료 여부와 관계없이 자바 프로세스가 종료된다.
- 단 셧다운 훅의 실행이 끝날 때까지 기다려 준다.
- 셧다운 훅의 실행이 끝나면 non 데몬 스레드의 실행 여부와 상관없이 자바 프로세스는 종료된다.
- 따라서 다른 스레드가 자원을 정리하거나 필요한 로그를 남길 수 있도록 셧다운 훅의 실행을 잠시 대기한다.

#### 🔍 서버 종료 결과
- 서버를 종료하면 `shutdown` 스레드가 `shutdownHook`을 실행하고, 세션의 `Socket`의 연결을 `close()`로 닫는다.
  - `[ Thread-0] java.net.SocketException: Socket closed`
  - `Session`의 `input.readUTF()`에서 입력을 대기하는 `Thread-0` 스레드는 `SocketException: Socket closed` 예외를 받고 종료된다.
    - 참고로 이 예외는 자신의 소켓을 닫았을 때 발생.


- `shutdown` 스레드는 서버 소켓을 `close()`로 닫는다.
  - `[   main] 서버 소켓 종료: java.net.SocketException: Socket closed`
  - `serverSocket.accept()`에서 대기하고 있던 `main` 스레드는 `java.net.SocketException: Socket closed` 예외를 받고 종료된다.

### 📌 정리
- **드디어 자원 정리까지 깔끔하게 해결한 서버 프로그램을 완성**

---

## ✅ 네트워크 예외 1 - 연결 예외
- 네트워크 연결 시 발생할 수 있는 예외들을 정리.

```java
public class ConnectMain {
    
    private static void unknownHostEx1() throws IOException {
        try {
            Socket socket = new Socket("999.999.999.999", 80);
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
    }
    
    private static void unknownHostEx2() throws IOException {
        try {
            Socket socket = new Socket("google.gogo", 80);
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
    }
    
    private static void connectionRefused() throws IOException {
        try {
          Socket socket = new Socket("localhost", 45678); // 미사용 포트
        } catch (ConnectException e) {
            // ConnectException: Connection refused
            e.printStackTrace();
        }
}
```

#### 🔍 java.net.UnknownHostException
- 호스트를 알 수 없음.
- `999.999.999.999`: 이런 IP는 존재하지 않는다.
- `google.gogo`: 이런 도메인 이름은 존재하지 않는다.

#### 🔍 java.net.ConnectException: Connection refused
- `Connection refused` 메시지는 연결이 거절되었다는 뜻이다.
- 연결이 거절되었다는 뜻은, 우선은 네트워크를 통해 해당 IP의 서버 컴퓨터에 접속은 했다는 뜻.
- 하지만 서버 컴퓨터가 45678 포트를 사용하지 않기 때문에 TCP 연결을 거절한다.
- IP에 해당하는 서버는 켜져 있지만, 사용하는 PORT가 없을 때 주로 발생.
- 네트워크 방화벽 등에서 무단 연결로 인지하고 연결을 막을 때도 발생한다.
- 서버 컴퓨터의 OS는 이때 TCP RST(Reset)라는 패킷을 보내서 연결을 거절한다.
- 클라이언트가 연결 시도 중에 RST 패킷을 받으면 이 예외가 발생한다.

#### 🔍 TCP RST(Reset) 패킷
- TCP 연결에 문제가 있다는 뜻. 이 패킷을 받으면 받은 대상은 바로 연결을 해제해야 한다.

---

# ⭐⭐ 매우 중요❗
## ✅ 네트워크 예외 2 - 타임아웃
- 네트워크 연결을 시도해서 서버 IP에 연결 패킷을 전달했지만 응답이 없는 경우 어떻게 될까?

### 📚 TCP 연결 타임아웃 - OS 기본
```java
public class ConnectTimeoutMain1 {
  public static void main(String[] args) {
    try {
        Socket socket = new Socket("192.168.1.250". 45678);
    } catch (ConnectException e) {
        e.printStackTrace();
    }
  }
}
```
- 사설 IP 대역(주로 공유기에서 사용하는 IP 대역)의 `192.168.1.250`을 사용했다.
- 해당 IP로 연결 패킷을 보내지만 IP를 사용하는 서버가 없으므로 TCP 응답이 오지 않는다.
- 또는 해당 IP로 연결 패킷을 보내지만 해당 서버가 너무 바쁘거나 문제가 있어서 연결 응답 패킷을 보내지 못하는 경우도 있다.
- 그렇다면 이때 무한정 기다려야 할까?

#### 🔍 OS 기본 대기 시간
- TCP 연결을 시도했는데 연결 응답이 없다면 OS에는 연결 대기 타임아웃이 설정되어 있다.
  - Windows: 약 21초
  - Linux: 약 75초에서 180초 사이
  - mac test 75초
- 해당 시간이 지나면 `java.net.ConnectException: Operation timed out`이 발생한다.
  - 윈도우의 경우 `java.net.ConnectException: Connection timed out: connect`가 발생, 예외는 같고, 오류 메시지가 다르다.


- TCP 연결을 클라이언트가 이렇게 오래 대기하는 것은 좋은 방법이 아니다.
- 연결이 안 되면 고객에게 빠르게 현재 연결에 문제가 있다고 알려주는 것이 더 나은 방법이다.

### 📚 TCP 연결 타임아웃 - 직접 설정
```java
public class ConnectTimeoutMain2 {
  public static void main(String[] args) {
    try {
        Socket socket = new Socket();
        socket.connect(new InetSocketAddress("192.168.1.250", 45678), 1000);
    } catch (ConnectException e) {
        e.printStackTrace();
    }
  }
}
```
#### 🔍 new Socket()
- `Socket` 객체를 생성할 때 인자로 IP, PORT를 모두 전달하면 생성자에서 바로 TCP 연결을 시도한다.
- 하지만 IP, PORT를 모두 빼고 객체를 생성하면, 객체만 생성되고, 아직 연결은 시도하지 않는다.
- 추가적으로 필요한 설정을 더 한 다음에 `socket.connect()`를 호출하면 그때 TCP 연결을 시도한다.
- 이 방식을 사용하면 추가적인 설정을 더 할 수 있는데, 대표적으로 **타임아웃**을 설정할 수 있다.
- `InetSocketAddress`: `SocketAddress`의 자식이다. IP, PORT 기반의 주소를 객체로 제공한다.
- `timeout`: 밀리초 단위로 연결 타임아웃을 지정할 수 있다.
  - 위 코드를 실행해 보면 설정한 시간인 1초가 지난 후 타임아웃 예외가 발생하는 것을 확인할 수 있다.

### 📚 TCP 소켓 타임아웃 - read 타임아웃
- 타임아웃 중에 또 하나 중요한 타임아웃이 있다.
- 바로 **소켓 타임아웃** 또는 **read 타임아웃**이라고 부르는 타임아웃이다.


- 앞에서 설명한 연결 타임아웃은 TCP 연결과 관련이 되어있다.
- 연결이 잘 된 이후에 클라이언트가 서버에 어떤 요청을 했다고 가정하자. 그런데 서버가 계속 응답을 주지 않는다면, 무한정 기다려야 할까?
- 서버에 사용자가 폭주하고 매우 느려져서 응답을 계속 주지 못하는 상황이라면 어떻게 해야 할까?
- **이런 경우에 사용하는 것이 바로 소켓 타임아웃(read 타임아웃)이다.**

```java
public class SoTimeoutServer {
  public static void main(String[] args) throws IOException, InterruptedException {
    ServerSocket serverSocket = new ServerSocket(12345);
    Socket socket = serverSocket.accept();
    
    Thread.sleep(1000000);
  }
}
```
- 서버는 소켓을 연결은 하지만, 아무런 응답을 주지 않는다.

```java
public class SoTimeoutClient {
  public static void main(String[] args) throws IOException, InterruptedException {
    Socket socket = new Socket("localhost", 12345);
    InputStream input = socket.getInputStream();
    try {
        socket.setSoTimeout(3000); // 타임아웃 시간 설정
        int read = input.read();
    } catch (Exception e) {
        e.printStackTrace();
    }
    
    socket.close();
  }
}
```
- `socket.setSoTimeout()`을 사용하면 밀리초 단위로 타임아웃 시간을 설정할 수 있다.
- 지정한 시간이 지나면 다음 예외가 발생한다.
  - `java.net.SocketTimeoutException: Read timed out`
- 타임아웃 시간을 설정하지 않으면 `read()`는 응답이 올 때까지 무한 대기한다.

#### 🤔 실무 이야기
- 실무에서 자주 발생하는 장애 원인 중 하나가 바로 연결 타임아웃, 소켓 타임아웃(read 타임아웃)을 누락하기 때문에 발생한다.
- **외부 서버와 통신을 하는 경우 반드시 연결 타임아웃과 소켓 타임아웃을 지정하자.**

---

## ✅ 네트워크 예외 3 - 정상 종료
TCP에는 2가지 종류의 경로가 있다.
- 정상 종료
- 강제 종료

#### 🔍 정상 종료
TCP에서 A, B가 서로 통신한다고 가정해 보자.

TCP 연결을 종료하려면 서로 FIN 메시지를 보내야 한다.
- A (FIN)-> B: A가 B로 FIN 메시지를 보낸다.
- A <-(FIN) B: FIN 메시지를 받은 B도 A에게 FIN 메시지를 보낸다.

`socket.close()`를 호출하면 TCP에서 종료의 의미인 FIN 패킷을 상대방에게 전달한다.

FIN 패킷을 받으면 상대방도 `socket.close()`를 호출해서 FIN 패킷을 상대방에게 전달해야 한다.
- 클라이언트와 서버가 연결되어 있다.
- 서버가 연결 종료를 위해 `socket.close()`를 호출한다.
  - 서버는 클라이언트에 FIN 패킷을 전달한다.
- 클라이언트는 FIN 패킷을 받는다.
  - 클라이언트의 OS에서 FIN에 대한 ACK 패킷을 전달한다.
- 클라이언트도 종료를 위해 `socket.close()`를 호출한다.
  - 클라이언트는 서버에 FIN 패킷을 전달한다.
  - 서버의 OS는 FIN 패킷에 대한 ACK 패킷을 전달한다.

각각의 상황에 따라 EOF를 해석하는 방법이 다르다.
- `read()` -> -1
  - EOF의 의미를 숫자 -1로 반환한다.
- `BufferedReader().readLine() -> null`
  - `BufferedReader()`는 문자 `String`을 반환한다. 따라서 -1을 표현할 수 없다. 대신에 `null`을 반환한다.
- `DataInputStream.readUTF()` -> `EOFException`
  - `DataInputStream`은 이 경우 `EOFException`을 던진다.
  - 예외를 통해서 연결을 종료할 수 있는 방법을 제공한다.

여기서 중요한 점은 EOF가 발생하면 상대방이 FIN 메시지를 보내면서 소켓 연결을 끊었다는 뜻이다.

이 경우 소켓에 다른 작업을 하면 안 되고, FIN 메시지를 받은 클라이언트도 `close()`를 호출해서 상대방에 FIN 메시지를 보내고 소켓 연결을 끊어야 한다.

---

## ✅ 네트워크 예외 4 - 강제 종료
TCP 연결 중에 문제가 발생하면 `RST`라는 패킷이 발생한다. 이 경우 연결을 즉시 종료해야 한다.

- 클라이언트와 서버가 연결되어 있다.
- 서버는 종료를 위해 `socket.close()`를 호출한다.
  - 서버는 클라이언트에 FIN 패킷을 전달한다.
- 클라이언트는 FIN 패킷을 받는다.
  - 클라리언트의 OS에서 FIN에 대한 ACK 패킷을 전달한다.
- 클라이언트는 `output.write(1)`를 통해 서버에 메시지를 전달한다.
  - 데이터를 전송하는 PUSH 패킷이 서버에 전달된다.
- 서버는 이미 FIN으로 종료를 요청했는데, PUSH 패킷으로 데이터가 전송되었다.
  - 서버가 기대하는 값은 FIN 패킷이다.
- 서버는 TCP 연결에 문제가 있다고 판단하고 즉각 연결을 종료하라는 RST 패킷을 클라이언트에 전송한다.

RST 패킷이 도착했다는 것은 현재 TCP 연결에 심각한 문제가 있으므로 해당 연결을 더는 사용하면 안 된다는 의미이다.

RST 패킷이 도착하면 자바는 `read()`로 메시지를 읽을 때 다음 예외를 던진다.
- MAC: `java.net.SocketException: Broken pipe`
- 윈도우: `java.net.SocketException: 현재 연결은 사용자의 호스트 시스템의 소프트웨어의 의해 중단되었습니다.`

#### 🔍 참고 - RST(Reset)
- TCP에서 RST 패킷은 연결 상태를 초기화(리셋)해서 더 이상 현재의 연결을 유지하지 않겠다는 의미를 전달한다.
  - 여기서 "Reset"은 현재의 세션을 강제로 종료하고, 연결을 무효화하라는 뜻이다.
- RST 패킷은 TCP 연결에 문제가 있는 다양한 상황에 발생한다. 예를 들어 다음과 같은 경우들이 있다.
  - TCP 스펙에 맞지 않는 순서로 메시지가 전달될 때
  - TCP 버퍼에 있는 데이터를 아직 다 읽지 않았는데, 연결을 종료할 때
  - 방화벽 같은 곳에서 연결을 강제로 종료할 때도 발생한다.

#### 🔍 참고 - java.netSocketException: Socket is closed
- 자기 자신의 소켓을 닫은 이후에 `read()`, `write()`을 호출할 때 발생한다.

### 📌 정리
- 상대방이 연결을 종료한 경우 데이터를 읽으면 EOF가 발생
  - -1, `null`, `EOFException` 등이 발생
  - 이 경우 연결을 끊어야 함.
- `java.net.SocketException: Connection reset`
  - RST 패킷을 받은 이후에 `read()` 호출
  - 윈도우 오류 메시지: `현재 연결은 사용자의 호스트 시스템의 소프트웨어의 의해 중단되었습니다`
- `java.net.SocketException: Broken pipe`
  - RST 패킷을 받은 이후에 `write()` 호출
  - 윈도우 오류 메시지: `현재 연결은 사용자의 호스트 시스템의 소프트웨어의 의해 중단되었습니다`
- `java.net.SocketException: Socket is closed`
  - 자신이 소켓을 닫은 이후에 `read()`, `write()` 호출

---

## 📌 네트워크 종료와 예외 정리
기본적으로 정상 종료, 강제 종료 모두 자원 정리하고 닫도록 설계하면 네트워크 예외가 발생하지 않는다.

예를 들어서 `SocketException`, `EOFException`은 모두 `IOException`의 자식이다.
따라서 `IOException`이 발생하면 자원을 정리하면 된다.
만약 더 자세히 분류해야 하는 경우가 발생하면 그때 예외를 구분해서 처리하면 된다. (특별한 경우에만)

---
