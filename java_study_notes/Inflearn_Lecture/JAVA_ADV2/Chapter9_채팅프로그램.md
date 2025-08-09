# 📚 Chapter 9: 채팅 프로그램

> 📌 공부 날짜: 2025/08/05
> - `JAVA `- 고급 2편
> - `References`: 인프런 - 김영한의 실전 자바

---

## ✅ 채팅 프로그램 설계

#### 🔍 채팅 프로그램 설계 - 클라이언트
사용자의 콘솔 입력과 서버로부터 메시지를 받는 부분을 별도의 스레드로 분리해야 한다.

#### 🔍 채팅 프로그램 설계 - 서버
채팅 프로그램의 핵심은 한 명이 이야기하면 그 이야기를 모두가 들을 수 있어야 한다.

따라서 하나의 클라이언트가 보낸 메시지를 서버가 받은 다음에, 서버에서 모든 클라이언트에게 메시지를 다시 전송해야 한다.

이렇게 하려면 서버에서 모든 세션을 관리해야 한다.

### 📚 채팅 프로그램 - 클라이언트
클라이언트는 다음 두 기능을 별도의 스레드에서 실행해야 한다.
- 콘솔의 입력을 받아서 서버로 전송한다.
- 서버로부터 오늘 메시지를 콘솔에 출력한다.

```java
public class ReadHandler implements Runnable {
    
    private final DataInputStream input;
    private final Client client;
    public boolean closed = false;
    
    public ReadHandler(DataInputStream input, Client client) {
        this.input = input;
        this.client = client;
    }
    
    @Override
    public void run() {
        try {
            while (true) {
                input received = input.readUTF();
                System.out.println(received);
            }
        } catch (IOException e) {
            log(e);
        } finally {
            client.close();
        }
    }
    
    public synchronized void close() {
        if (closed) return;
        
        closed = true;
        log("readHandler 종료");
    }
}
```
- 중복 종료를 막기 위해 동기화 코드와 `closed` 플래그를 사용했다.
- `IOException` 예외가 발생하면 `client.close()`를 통해 클라이언트를 종료하고, 전체 자원을 정리한다.

```java
public class WriteHandler implements Runnable {
    
    private static final String DELIMITER = "|";
    
    private final DataOutputStream output;
    private final Client client;
    
    private boolean closed = false;
    
    public WriteHandler(DataOutputStream output, Client client) {
        this.output = output;
        this.client = client;
    }
    
    @Override
    public void run() {
        Scanner sc = new Scanner(System.in);
        try {
            String username = inputUserName(sc);
            output.writeUTF("/join" + DELIMITER + username);
            
            while (true) {
                String toSend = sc.nextLine();
                if (toSend.isEmpty()) continue;
                
                if (toSend.equals("exit")) {
                    output.writeUTF(toSend);
                    breka;
                }
                
                if (toSend.startsWith("/")) output.writeUTF(toSend);
                else output.writeUTF("/message" + DELIMITER + toSend);
            }
        } catch (IOException | NoSuchElementException e) {
            log(e);
        } finally {
            client.close();
        }
    }
    
    private static String inputUserName(Scanner sc) {
      System.out.println("이름을 입력하세요.");
      String username;
      do {
          username = sc.nextLine();
      } while (username.isEmpty());
      return username;
    }
}

    public synchronized void close() {
        if (closed) return;
        
        try {
            System.in.close();
        } catch (IOException e) {
            log(e);
        }
        closed = true;
        log("WriteHandler 종료");
    }
```
`close()`를 호출하면 `System.in.close()`를 통해 사용자의 콘솔 입력을 닫는다.
이렇게 하면 `Scanner`를 통한 콘솔 입력인 `sc.nextLine()` 코드에서 대기하는 스레드에 다음 예외가 발생하면서, 대기 상태에서 빠져나올 수 있다.

`java.util.NoSuchElementException: No line found`

서버가 연결을 끊은 경우에 클라이언트의 자원이 정리되는데, 이때 유용하게 사용된다.

`IOException` 예외가 발생하면 `client.close()`를 통해 클라이언트를 종료하고, 전체 자원을 정리한다.

#### 🔍 윈도우 OS 추가
윈도우의 경우 `System.in.close()`를 호출해도 사용자의 콘솔 입력이 닫히지 않는다.
사용자가 어떤 내용이든 입력을 해야 그다음에 `java.util.NoSuchElement: No line found` 예외가 발생하면서 대기 상태에서 빠져나올 수 있다.

```java
public class Client {
    
    private final String host;
    private final int port;
    
    private Socket socket;
    private DataInputStream input;
    private DataOutputStream output;
    
    private ReadHandler readHandler;
    private WriteHandler writeHandler;
    private boolean closed = false;
    
    public Client(String host, int port) {
        this.host = host;
        this.port = port;
    }
    
    public void start() throws IOException {
        log("클라이언트 시작");
        socket = new Socket(host, port);
        input = new DataInputStream(socket.getInputStream());
        output = new DataOutputStream(socket.getOutputStream());
        
        readHandler = new ReadHandler(input, this);
        writeHandler = new WriteHanlder(output, this);
        Thread readThread = new Thread(readHandler, "readHandler");
        Thread writeThread = new Thread(writeHandler, "writeHandler");
        readThread.start();
        writeThread.start();
    }
    
    public synchronized void close() {
        if (closed()) return;
        
        writeHandler.close();
        readHandler.close();
        closeAll(socket, input, output);
        closed = true;
        log("연결 종료: " + socket);
    }
}
```
- 클라이언트 전반을 관리하는 클래스이다.

```java
public class ClientMain {
    
    private static final int PORT = 12345;

    public static void main(String[] args) {
        Client client = new Client("localhost", PORT);
        client.start();
    }
}
```

---

### 📚 채팅 프로그램 - 서버 1
```java
public class Session implements Runnable {
    
    private final Socket socket;
    private final DataInputStream input;
    private final DataOutputStream output;
    private final CommandManager commandManager;
    private final SessionManager sessionManager;
    
    private boolean closed = false;
    private String username;
    
    public Session(Socket socket, CommandManager commandManager, SessionManager sessionManager) throws IOException {
        this.socket = socket;
        this.input = new DataInputStream(socket.getInputStream());
        this.output = new DataOutputStream(socket.getOutputStream());
        
        this.commandManager = commandManager;
        this.sessionManager = sessionManager;
        this.sessionManager.add(this);
    }

    @Override
    public void run() {
        try {
            while (true) {
                String recevied = input.readUTF();
                commandManger.execute(received, this);
            }
        } catch (IOException e) {
            log(e);
        } finally {
            sessionManager.remove(this);
            sessionManager.sendAll(username + "님이 퇴장했습니다.");
            close();
        }
    }

    public void send(String message) throws IOExcpetion {
        output.writeUTF(message);
    }
    
    public synchronized void close() {
        if (closed) return;
        
        closeAll(socket, input, output);
        closed = true;
        log("연결 종료: " + socket);
    }
    
    public String getUsername() {
        return username;
    }
    
    public void setUsername(String username) {
        this.username = username;
    }
    
}
```
- `CommandManager`는 명령어를 처리하는 기능을 제공.

```java
public class SessionManager {
    
    private final List<Session> sessions = new ArrayList<>();
    
    public synchronized void add(Session session) {
        sessions.add(session);
    }
    
    public synchronized void remove(Session session) {
        sessions.remove(session);
    }
    
    public synchronized void closeAll() {
        for (Session session : sessions) {
            session.close();
        }
        sessions.clear();
    }
    
    public synchronized void sendAll(String message) {
        for (Session session : sessions) {
            try {
                session.send(message);
            } catch (IOException e) {
                log(e);
            }
        }
    }
    
    public synchronized List<String> getAllUsername() {
        List<String> usernames = new ArrayList<>();
        for (Session session : sessions) {
            if (session.getUsername() != null) {
                usernames.add(session.getUsername());
            }
        }
        return usernamses;
    }
}
```

```java
public class Server {
    
    private final int port;
    private final CommandManager commandManager;
    private final SessionManager sessionManager;
    
    private ServerSocket serverSocket;
    
    public Server(int port, CommandManager commandManager, SessionManager sessionManager) {
        this.port = port;
        this.commandManager = commandManager;
        this.sessionManager = sessionManager;
    }
    
    public void start() throws IOException {
        log("서버 시작");
        serverSocket = new ServerSocket(port);
        
        addShutdownHook();
        running();
    }
    
    private void addShutdownHook() {
        ShutdownHook target = new ShutdownHook(serverSocket, sessionManager);
        Runtime.getRuntime().addShutdownHook(new Thread(targert, "shutdown"));
    }
    
    private void running() {
        try {
            while (true) {
                Socket socket = serverSocket.accpet();
                Session session = new Session(socket, commandManager, sessionManager);
                Thread thread = new Thread(session);
                thread.start();
            }
        } catch (IOException e) {
            log(e);
        }
    }
    
    static class ShutdownHook implements Runnable {
        
        private final ServerSocket serverSocket;
        private final SessionManager sessionManager;
        
        public ShutdownHook(ServerSocket serverSocket, SessionManager sessionManager) {
            this.serverSocket = serverSocket;
            this.sessionManager = sessionManager;
        }
        
        @Override
        public void run() {
            try {
                sessionManager.closeAll();
                serverSocket.close();
                
                Thread.sleep(1000);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
public class ServerMain {
    
    private static final int PORT = 12345;

    public static void main(String[] args) {
        SessionManager sessionManager = new SessionManager();
        
        CommandManager commandManager = new CommandManagerV4(sessionManager);
        
        Server server  = new Server(PORT, commandManager, sessionManager);
        server.start();
    }
}
```

```java
public class CommandManagerV4 implements CommandManager {
    
    public static final String DELIMITER = "\\|";
    private final Map<String, Command> commands = new HashMap<>();
    private final Command defaultCommand = new DefaultCommand();
    
    public CommandManagerV3(SessionManager sessionManager) {
        commands.put("/join", new JoinCommand(sessionManager));
        commands.put("/message", new MessageCommand(sessionManager));
        commands.put("/change", new ChangeCommand(sessionManager));
        commands.put("/users", new UsersCommand(sessionManager));
        commands.put("/exit", new ExitCommand());
    }
    
    @Override
    public void execute(String totalMessage, Session session) throws IOException {
        String[] args = totalMessage.split(DELIMITER);
        String key = args[0];
        
        Command command = commands.getOrDefault(key, defaultCommand);
        command.execute(args, session);
    }
}
```
#### Map<String, Command> commands
- 명령어는 Map에 보관한다. 명령어 자체를 Key로 사용하고, 각 Key에 해당하는 `Command` 구현체를 저장해둔다.

#### execute()
- `Command command = commands.get(key)`
- 명령어(key)를 처리할 `Command` 구현체를 `commands`에서 찾아서 실행한다.


#### 🔍 참고 - 동시성과 읽기
- 여러 스레드가 `commands = new HashMap<>()`을 동시에 접근해서 데이터를 조회한다.
  - 하지만 `commands`는 데이터 초기화 이후에는 데이터를 전혀 변경하지 않는다.
  - 따라서 여러 스레드가 동시에 값을 조회해도 문제가 발생하지 않는다. 만약 `commands`의 데이터를 중간에 변경할 수 있게 하려면 동시성 문제를 고민해야 한다.


- `Map`에는 `getOrDefault(key, defaultObject)`라는 메서드가 존재한다.
- 만약 key를 사용해서 객체를 찾을 수 있다면 찾고, 찾을 수 없다면 옆에 있는 `defaultObject`를 반환한다.
- 이 기능을 사용하면 `null`을 받지 않고 항상 `Command` 객체를 받아서 처리할 수 있다.

---

### 📚 Null Object Pattern
이와 같이 `null`을 객체(Object)처럼 처리하는 방법을 Null Object Pattern 이라 한다.

이 디자인 패턴은 `null` 대신 사용할 수 있는 특별한 객체를 만들어, `null`로 인해 발생할 수 있는 예외 상황을 방지하고 코드의 간결성을 높이는 데 목적이 있다.

Null Object Pattern을 사용하면 `null` 값 대신 특정 동작을 하는 객체를 반환하게 되어, 클라이언트 코드에서
`null` 체크를 할 필요가 없어진다. 이 패턴은 코드에서 불필요한 조건문을 줄이고 객체의 기본 동작을 정의하는 데 유용하다.

---

### 📚 Command Pattern
커맨드 패턴은 디자인 패턴 중 하나로, 요청을 독립적인 객체로 변환해서 처리한다.

커맨드 패턴의 특징
- 분리: 작업을 호출하는 객체와 작업을 수행하는 객체를 분리한다.
- 확장성: 기존 코드를 변경하지 않고 새로운 명령을 추가할 수 있다.

#### 커맨드 패턴의 장점
- 이 패턴의 장점은 새로운 커맨드를 쉽게 추가훌 수 있다는 점이다. 예를 들어, 새로운 커맨드를 추가하고 싶다면, 새로운 `Command`의 구현체만 만들면 된다.
그리고 기존 코드를 대부분 변경할 필요가 없다.
- 작업을 호출하는 객체와 작업을 수행하는 객체가 분리되어 있다.
- 각각의 기능이 명확하게 분리된다. 개발자가 어떤 기능을 수정해야 할 때, 수정해야 하는 클래스가 아주 명확해진다.

#### 커맨드 패턴의 단점
- **복잡성 증가**: 간단한 작업을 수행하는 경우에도 `Command` 인터페이스와 구현체들, `Command` 객체를 호출하고 관리하는 
클래스 등 여러 클래스를 생성해야 하기 때문에 코드의 복잡성이 증가할 수 있다.
- 모든 설계에는 트레이드 오프가 있다. 예를 들어 단순한 if문 몇 개로 해결할 수 있는 문제에 복잡한 커맨드 패턴을 도입하는 것은 좋은 설계가 아닐 수 있다.
- 기능이 어느 정도 있고, 각각의 기능이 명확하게 나누어질 수 있고, 향후 기능의 확장까지 고려해야 한다면 커맨드 패턴을 좋은 대안이 될 수 있다.

---