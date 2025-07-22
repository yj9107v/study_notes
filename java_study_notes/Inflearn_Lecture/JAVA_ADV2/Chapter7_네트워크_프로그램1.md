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

