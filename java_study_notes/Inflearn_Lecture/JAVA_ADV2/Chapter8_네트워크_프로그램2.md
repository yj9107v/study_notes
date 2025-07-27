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
    private final boolean closed = false;
    
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
