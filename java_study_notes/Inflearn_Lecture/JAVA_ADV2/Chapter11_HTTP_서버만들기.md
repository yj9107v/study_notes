# 📚 Chapter 11: HTTP 서버 만들기

> 📌 공부 날짜: 2025/08/24
> - `JAVA `- 고급 2편
> - `References`: 인프런 - 김영한의 실전 자바

---

## ✅ HTTP 서버 1 - 시작
- 웹 브라우저에서 우리 서버에 접속하면 다음과 같은 HTML을 응답하는 것이다.
```html
<h1>Hello World</h1>
```

```java
public class HttpServerV1 {
    private final int port;
    
    public HttpServerV1 {
        this.port = port;
    }
    
    public void start() throws IOException {
        ServerSocket serverSocket = new ServerSocket(port);
        
        while (true) {
            Socket socket = serverSocket.accpet();
            process(socket);
        }
    }
    
    private void process(Socket socket) throws IOException {
        try (socket;
             BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()), UTF_8);
             PrintWriter writer = new PrintWriter(socket.getOutputStream(), false, UTF_8)) {
            
            String requestString = requestToString(reader);
            if (requestString.contains("/favicon.ico")) {
                log("favicon 요청");
                return;
            }
            log("HTTP 요청 정보 출력");
            System.out.println(requestString);
            
            log("HTTP 응답 생성 중...");
            responseToClient(writer);
            log("HTTP 응답 전달 완료");
        }
    }
    
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
    
    private void responseToClient(PrintWriter writer) throws IOException {
        // 웹 브라우저에 전달하는 내용
      
        String body = "<h1>Hello World</h1>";
        int length = body.getBytes(UTF_8).length;
        
        StringBuilder sb = new StringBuilder();
        sb.append("HTTP/1.1 200 OK\r\n");
        sb.append("Content-Type: text/html\r\n");
        sb.append("Content-Length: ").append(length).append("\r\n");
        sb.append("\r\n"); // header, body 구분 라인
        sb.append(body);
        
        log("HTTP 응답 정보 출력");
        System.out.println(sb);
        
        writer.println(sb);
        writer.flush();
    }
}
```

- HTTP 메시지의 주요 내용들은 문자로 읽고 쓰게 된다.
- 따라서 여기서는 `BufferedReader`, `PrintWriter`를 사용했다.
- `Stream`을 `Reader`, `Writer`로 변경할 때는 항상 인코딩을 확인하자.

#### 🔍 AutoFlush
- `new PrintWriter(socket.getOutputStream(), false, UTF_8)`
  - `PrintWriter`의 두 번째 인자는 `autoFlush()` 여부이다.
  - 이 값을 `true`로 설정하면 `println()`으로 출력할 때마다 자동으로 플러시 된다.
    - 첫 내용을 빠르게 전송할 수 있지만, 네트워크 전송이 자주 발생한다.
  - 이 값을 `false`로 설정하면 `flush()`를 직접 호출해 주어야 데이터를 전송한다.
    - 데이터를 모아서 전송하므로 **네트워크 전송 횟수를 효과적으로 줄일 수 있다**. 한 패킷에 많은 양의 데이터를 담아서 전송할 수 있다.

#### 🔍 requestToString()
- HTTP 요청을 읽어서 `String`으로 반환한다.
- HTTP 요청의 시작 라인, 헤더까지 읽는다.

#### 🔍 /favicon.ico
- 웹 브라우저에서 해당 사이트의 작은 아이콘을 추가로 요청할 수 있다. 여기서는 사용하지 않으므로 무시한다.

#### 🔍 responseToClient()
- HTTP 응답 메시지를 생성해서 클라이언트에 전달한다.
- 시작라인, 헤더, HTTP 메시지 바디를 전달한다.
- HTTP 공식 스펙에서 다음 라인은 `\r\n(캐리지 리턴 + 라인 피드)`로 표현한다. 참고로 `\n`만 사용해도 대부분의 웹 브라우저는 문제없이 작동한다.
- 마지막에 `writer.flush()`를 호출해서 데이터를 전송한다.

---
---

### 📚 HTTP 요청 메시지
```text
GET / HTTP/1.1
Host: localhost:12345
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) ...
Accept: text/html, application/xhtml+xml, application/xml;q=0.9...
Accept-Encoding: gzip, deflate...
Accept-Language: ko, en;q=0.9...
```
#### 🔍 시작 라인(start line)
- `GET`: GET 메서드 (조회)
- `/`: 요청 경로, 별도의 요청 경로가 없으면 `/`를 사용한다.
- `HTTP/1.1`: HTTP 버전

#### 🔍 헤더(header)
- `Host`: 접속하는 서버 정보
- `User-Agent`: 웹 브라우저의 정보
- `Accept`: 웹 브라우저가 전달받을 수 있는 HTTP 응답 메시지 바디 형태
- `Accpet-Encoding`: 웹 브라우저가 전달받을 수 있는 인코딩 형태
- `Accept-Language`: 웹 브라우저가 전달받을 수 있는 언어 형태

---
---

#### 📚 HTTP 응답 메시지
```text
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 20

<h1>Hello World</h1>
```

#### 🔍 시작 라인
- `HTTP/1.1`: HTTP 버전
- `200`: 성공
- `OK`: `200`에 대한 설명

#### 🔍 헤더
- `Content-Type`: HTTP 메시지 바디의 데이터 형태, 여기서는 HTML을 사용
- `Content-Length`: HTTP 메시지 바디의 데이터 길이

#### 🔍 바디
- `<h1>Hello World</h1>`

---
---
#### 🤔 문제
- 서버는 동시에 수많은 사용자의 요청을 처리할 수 있어야 한다.
- 현재 서버는 한 번에 하나의 요청만 처리할 수 있다는 문제가 있다.
- 다른 웹 브라우저 2개를 동시에 열어서 사이트를 실행할 수 있어야 한다. (예를 들어서 크롬, 엣지, 사파리 등 다른 브라우저를 열어야 확실한 테스트 가능)
- 첫 번째 요청이 모두 처리되고 나서 두 번째 요청이 처리되는 것을 확인할 수 있다.

---

## ✅ HTTP 서버 2 - 동시 요청
```java
public class HttpRequestHandlerV2 implements Runnable {
    private final Socket socket;
    
    public HttpRequestHandlerV2(Socket socket) {
        this.socket = socket;
    }
    
    @Override
    public void run() {
        try {
            process();
        } catch (Exception e) {
            log(e);
        }
    }
    
    private void process() throws IOException {}
  
    private String requestToString(BufferedReader reader) throws IOException {}
  
    private void responseToClient(PrintWriter writer) {}
}
```
- 클라이언트의 `HttpRequestHandler`는 이름 그대로 클라이언트가 전달한 HTTP 요청을 처리한다.
- 동시에 요청한 수만큼 별도의 스레드에서 `HttpRequestHandler`가 수행된다.

```java
public class HttpServerV2 {
    
    private final ExecutorService es = Executors.newFixedThreadPool(10);
    private final int port;
    
    public HttpServerV2(int port) {
        this.port = port;
    }
    
    public void start() throws IOException {
        ServerSocket serverSocket = new ServerSokcet(port);
        
        while (true) {
            Socket socket = serverSocket.accept();
            es.submit(new HttpRequestHandlerV2(socket));
        }
    }
}
```
- `ExecutorService`: 스레드 풀을 사용한다. 여기서는 `newFixedThreadPool(10)`을 사용해서 최대 동시에 10개의 스레드를 사용할 수 있도록 했다. 결과적으로 10개의 요청을 동시에 처리할 수 있다.
- 참고로 실무에서는 상황에 따라 다르지만 보통 수백 개 정도의 스레드를 사용한다.
- `es.submit(new HttpRequestHandlerV2(socket))`: 스레드 풀에 `HttpRequestHandlerV2` 작업을 요청한다.
- 스레드 풀에 있는 스레드가 `HttpRequestHandlerV2`의 `run()`을 수행한다.

---

## ✅ HTTP 서버 3 - 기능 추가
- HTTP 서버들은 URL 경로를 사용해서 각각의 기능을 제공한다.

```java
public class HttpRequestHandlerV3 implements Runnable {
    private final Socket socket;
    
    public HttpRequestHandlerV3(Socket socket) {
        this.socket = socket;
    }
    
    @Override
    public void run() {
        try {
            process(socket);
        } catch (Exception e) {
            log(e);
        }
    }
    
    private void process(Socket socket) throws IOException {
        try (socket;
             BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream(), UTF_8));
             PrintWriter writer = new PrintWriter(new OutputStreamWriter(socket.getOutputStream(), false, UTF_8))) {

            String requestString = requestToString(reader);
            if (requestString.contains("/favicon.ico")) {
                log("favicon 요청");
                return;
            }
            System.out.println(requestString);

            if (requestString.startsWith("GET /site1")) {
                site1(writer);
            } else if (requestString.startsWith("GET /site2")) {
                site2(writer);
            } else if (requestString.startsWith("GET /search")) {
                search(writer, requestString);
            } else if (requestString.startsWith("GET / ")) {
                home(writer);
            } else {
                notFound(writer);
            }
            log("HTTP 응답 전달 완료");
        }
    }
    
    private String requestToString(BufferedReader reader) {}
  
    private static void home(PrintWriter writer) {
        // 원칙적으로 Content-Length를 계산해서 전달해야 하지만, 여기선 생략함.
      
        writer.println("HTTP/1.1 200 OK");
        writer.println("Content-Type: text/html; charset=UTF-8");
        writer.println();
        writer.println("<h1>home</h1>");
        writer.println("<ul>");
        writer.println("<li><a href='/site1>site1</a></li>");       
        writer.println("<li><a href='/site2>site2</a></li>");       
        writer.println("<li><a href='/search/?q=hello>검색</a></li>");       
        writer.println("</ul>");
        writer.flush();
    }
    
    private static void site1(PrintWriter writer) {}
  
    private static void site2(PrintWriter writer) {}
    
    private static void notFound(PrintWriter writer) {}
  
    private static void search(PrintWriter writer, String requestString) {}
  
}
```
- HTTP 요청 메시지의 시작 라인을 파싱하고, 요청 URL에 맞추어 응답을 전달한다.
  - `GET /` -> `home()` 호출
  - `GET /site1` -> `site1()` 호출
- 응답 시 원칙적으로 헤더에 메시지 바디의 크기를 계산해서 `Content-Length`를 전달해야 하지만, 생략했음.

#### 🔍 검색
검색 시 다음과 같은 형식으로 요청이 온다.
- `GET /search?q=hello`
- URL에서 `?` 이후의 부분에 `key1=value1&key2=value2` 포맷으로 서버에 데이터를 전달할 수 있다.
- 이 부분을 파싱하면 요청하는 검색어를 알 수 있다.

---

## ✅ URL 인코딩
### 📚 URL이 ASCII를 사용하는 이유
- HTTP 메시지에서 시작 라인(URL을 포함)과 HTTP 헤더의 이름은 항상 ASCII를 사용해야 한다.
- HTTP 메시지 바디는 UTF-8과 같은 다른 인코딩을 사용할 수 있다.

#### 🔍 HTTP에서 URL이 ASCII 문자를 사용하는 이유
- 인터넷이 처음 설계되던 시기(1980~1990년대)에, 대부분의 컴퓨터 시스템은 ASCII 문자 집합을 사용했다.
- 전 세계에서 사용하는 다양한 컴퓨터 시스템과 네트워크 장비 간의 호환성을 보장하기 위해, URL은 단일한 문자 인코딩 체계를 사용해야 했다.
- HTTP URL이 ASCII만을 지원하는 이유는 초기 인터넷의 기술적 제약과 전 세계적인 호환성을 유지하기 위한 선택이다.
- HTTP 스펙은 매우 보수적이고, 호환성을 가장 우선 시 한다.

---
---
- 그렇다면 검색어로 사용하는 `/search?q=hello`를 사용할 때 `q=가나다`와 같이 URL에 한글을 전달하려면 어떻게 해야 할까?

### 📚 퍼센트(%) 인코딩
- 한글을 UTF-8 인코딩으로 표현하면 한 글자에 3byte의 데이터를 사용한다.
- 가, 나, 다를 UTF-8 인코딩의 16진수로 표현하면 다음과 같다.
  - 가: EA,B0,80 (3byte)
  - 나: EB,82,98 (3byte)
  - 다: EB,8B,A4 (3byte)

  
- URL은 ASCII 문자만 표현할 수 있으므로, UTF_8 문자를 표현 X
- 그래서 한글 "가"를 예를 들면 UTF-8 16진수로 표현한 각각의 바이트 문자 앞에 %(퍼센트)를 붙이는 것이다.
  - `q=가`
  - `q=%EA%B0%80`


- 이렇게 하면 ASCII 문자를 사용해서 16진수로 표현된 UTF-8을 표현할 수 있다.
- 이렇게 각각의 16진수 byte를 문자로 표현하고, 해당 문자 앞에 `%`를 붙이는 것을 **퍼센트(`%`) 인코딩**이라 한다.


- `%` 인코딩 후에 클라이언트에서 서버로 데이터를 전달하면 서버는 각각의 `%`를 제거하고, `EA`, `B0`, `80`이라는 각 문자를 얻는다.
  - 그리고 이렇게 얻은 문자를 16진수 byte로 변경한다. 이 3개의 byte를 모아서 UTF-8로 디코딩하면 "가"라는 글자를 얻을 수 있다.

#### 🔍 % 인코딩
- 자바가 제공하는 `URLEncoder.encode()`, `URLDecoder.decode()`를 사용하면 `%`인코딩, 디코딩을 처리할 수 있다.

#### 🤔 '%' 인코딩 정리
- `%` 인코딩은 데이터 크기에서 보면 효율이 떨어진다. 문자 "가"는 단지 3byte만 필요하다. 그런데 % 인코딩을 사용하면 무려 9byte가 사용된다.
- HTTP는 매우 보수적이다. 호환성을 최우선에 둔다.

---

## ✅ HTTP 서버 4 - 요청, 응답
- HTTP 요청 메시지와 응답 메시지는 규칙이 있으므
- 로, 각 규칙에 맞추어 객체로 만들면, 단순히 `String` 문자로 다루는 것보다 훨씬 더 구조적이고 객체지향적인 편리한 코드를 만들 수 있다.

#### 🔍 HTTP 요청 메시지
```java
public class HttpRequest {
    
    private String method;
    private String path;
    private final Map<String, String> queryParameters = new HashMap<>();
    private final Map<String, String> headers = new HashMap<>();
    
    public HttpRequest(BufferedReader reader) throws IOException {
        parseRequestLine(reader);
        paresHeaders(reader);
    }
    
    private void parseRequestLine(BufferedReader reader) throws IOException {
        String requestLine = reader.readLine();
        if (requestLine == null) {
            throw new IOException("EOF: No request line received");
        }
        
        String[] parts = requestline.split(" ");
        if (parts.length != 3) {
            throw new IOException("Invalid request line: " + requestLine);
        }
        
        method = parts[0];
        String[] pathParts = parts[1].split("\\?");
        path = pathParts[0];
        
        if (pathParts.length > 1) {
            parseQueryParameters(pathParts[1]);
        }
    }
    
    private void parseQueryParameters(String queryString) {
        for (String param : queryString.split("&")) {
            String[] keyValue = param.split("=");
            String key = URLDecoder.decode(keyValue[0], UTF_8);
            String value = keyValue.length > 1 ? URLDecoder.decode(keyValue[1], UTF_8) : "";
            queryParameters.put(key, value);
        }
    }
    
    private void parseHeaders(BufferedReader reader) throws IOException {
        String line;
        while (!(line = reader.readLine()).isEmpty()) {
            String[] headerParts = line.split(":");
            //trim() 앞 뒤에 공백 제거
            headers.put(headerParts[0].trim(), headerParts[1].trim());
        }
    }
    
    public String getMethod() {
        return method;
    }
    
    public String getPath() {
        return path;
    }
    
    public String getParameter(String name) {
        return queryParameters.get(name);
    }
    
    public String getHeader(String name) {
        return headers.get(name);
    }
}
```
- `reader.readLine()`: 클라이언트가 연결만 하고 전송 없이 연결을 종료하는 경우 `null`이 반환된다.
  - 이 경우 간단히 `throw new IOException(EOF)` 예외를 던지겠다.

- 시작 라인을 통해 `method`, `path`, `queryParameters`를 구할 수 있다.
  - method: `GET`
  - path: `/search`
  - queryParameters: `[q=hello]`
- `query`, `header`의 경우 `key=value` 형식이기 때문에 `Map`을 사용하면 이후에 편리하게 데이털르 조회할 수 있다.


- 퍼센트 디코딩도 `URLDecoder.decode()`를 사용해서 처리한 다음에 `Map`에 보관한다.
  - 따라서 `HttpRequest` 객체를 사용하는 쪽에서는 퍼센트 디코딩을 고민하지 않아도 된다.


- HTTP 명세에서 헤더가 끝나는 부분을 빈 라인으로 구분한다.
  - `while(!(line = reader.readLine()).isEmpty())`
- 이렇게 하면 시작 라인의 다양한 정보와 헤더를 객체로 구조화할 수 있다.

#### 🔍 HTTP 응답 메시지
```java
public class HttpResponse {
    private final PrintWriter writer;
    private int statusCode = 200;
    private final StringBuilder bodyBuilder = new StringBuilder();
    private String contentType = "text/html; charset=UTF-8";
    
    public HttpResponse(PrintWriter writer) {
        this.writer = writer;
    }
    
    public void setStatusCode(int statusCode) {
        this.statusCode = statusCode;
    }
    
    public void setContentType(String contentType) {
        this.contentType = contentType;
    }
    
    public void writeBody(String body) {
        bodyBuilder.append(body);
    }
    
    public void flush() {
        int contentLength = bodyBuilder.toString().getBytes(UTF_8).length;
        writer.println("HTTP/1.1 " + statusCode + " " + getReasonPharse(statusCode));
        writer.println("Content-Length: " + contentLength);
        writer.println();
        writer.println(bodyBuilder);
        writer.println();
    }
    
    private String getReasonPharse(int statusCode) {
        switch (statusCode) {
          case 200:
              return "OK";
          case 404:
              return "Not Found";
          case 500:
              return "Internal Server Error";
          default:
              return "Unknown Status";
        }
    }
}
```
- 클라이언트의 요청이 오면 요청 정보를 기반으로 `HttpRequest` 객체를 만들어둔다. 이때 `HttpResponse`도 함께 만든다.

#### 🤔 정리
- `HttpRequest`, `HttpResponse` 객체가 HTTP 요청과 응답을 구조화한 덕분에 많은 중복을 제거하고, 또 코드도 매우 효과적으로 리팩토링 할 수 있었다.
- 전체적인 코드가 2가지로 분류된다.
  - HTTP 서버와 관련된 부분
    - `HttpServer`, `HttpRequestHandler`, `HttpRequest`, `HttpResponse`
  - 서비스 개발을 위한 로직
    - `site1()`, `site2()`, `search()`, `notFound()`, `home()`

  
- 만약 웹을 통한 회원 관리 프로그램 같은 서비스를 새로 만들어야 한다면, 기존 코드에서 HTTP 서버와 관련된 부분을 거의 재사용하고, 서비스 개발을 위한 로직만 추가하면 된다.

---

## ✅ HTTP 서버 5 - 커맨드 패턴
- HTTP 서버와 관련된 부분을 본격적으로 구조화해보자. 서비스 개발을 위한 로직과 명확하게 분리해 보자.
- 여기서 핵심은 HTTP 서버와 관련된 부분을 코드 변경 없이 재사용 가능해야 한다는 점이다.

### 📚 커맨드 패턴 도입
- 커맨드 패턴을 도입하면 좋을 것 같은 코드.
```java
if (request.getPath().equals("/site1")) {
    site1(response);
} else if (request.getPath().equals("/site2")) {
    site2(response);
} else if (request.getPath().equals("/search")) {
    search(response, request);
} else if (request.getPath().equals("/")) {
    home(response);
} else {
    notFound(response);
}
```

- 커맨드 패턴을 사용하면 확장성이라는 장점도 있지만, HTTP 서버와 관련된 부분과 서비스 개발을 위한 로직을 분리하는데도 도움이 된다.

```java
public interface HttpServlet {
    void service(HttpRequest request, HttpResponse response) throws IOException;
}
```
- `HttpServlet`이라는 이름의 인터페이스를 만들었다.
  - HTTP, Server, Applet의 줄임말이다. (HTTP 서버에서 실행되는 작은 자바 프로그램(애플릿))
- 이 인터페이스의 `service()` 메서드가 있는데, 여기에 서비스 개발과 관련된 부분을 구현하면 된다.


- HTTP 서블릿을 구현해서 서비스의 각 기능을 구현해 보자.

---
---
### 📚 서비스 서블릿
```java
public class HomeServlet implements HttpServlet {
    @Override
    public void service(HttpRequest request, HttpResponse response) throws IOException {
      response.writeBody("<h1>home</h1>");
      response.writeBody("<ul>");
      response.writeBody("<li><a href='/site1>site1</a></li>");
      response.writeBody("<li><a href='/site2>site2</a></li>");
      response.writeBody("<li><a href='/search/?q=hello>검색</a></li>");
      response.writeBody("</ul>");
    }
}
```
- 이런 식으로 기능을 다 도출해서 서블릿을 만든다.
- `HomeServlet`, `Site1Servlet`, `Site2Servlet`, `SearchServlet`는 현재 프로젝트에서만 사용하는 개별 서비스를 위한 로직이다.

---
---
### 📚 공용 서블릿
- `NotFoundServlet`, `InternalErrorServlet`, `DiscardServlet`은 여러 프로젝트에서 공용으로 사용하는 서블릿이다.
  - 따라서 위 개별 서비스 서블릿과 패키지를 분리한다.


- `HttpServlet`을 관리하고 실행하는 `ServletManager` 클래스도 만들자.
```java
public class ServletManager {
    private final Map<String ,HttpServlet> servletMap = new HashMap<>();
    private HttpServlet defaultServlet;
    private HttpServlet notFoundErrorServlet = new NotFouundServlet();
    private HttpServlet internalErrorServlet = new InternalErrorServlet();
    
    public ServletManager() {}
  
    public void add(String path, HttpServlet servlet) {
        servletMap.put(path, servlet);
    }
    
    public void setDefaultServlet(HttpServlet defaultServlet) {
        this.defaultServlet = defaultServlet;
    }
    
    public void setNotFoundErrorServlet(HttpServlet notFoundErrorServlet) {
        this.notFoundErrorServlet = notFoundErrorServlet;
    }
    
    public void setInternalErrorServlet(HttpServlet internalErrorServlet) {
        this.internalErrorServlet = internalErrorServlet;
    }
    
    public void execute(HttpRequest request, HttpResponse response) throws IOException {
        try {
            HttpServlet servlet = servletMap.getOrDefault(request.getPath(), defaultServlet);
            if (servlet == null) {
                throw new PageNotFoundException("request url= " + request.getPath());
            }
            servlet.service(request, response);
        } catch (PageNotFoundException e) {
            e.printStackTrace();
            notFoundErrorServlet.service(request, response);
        } catch (Exception e) {
            e.printStackTrace();
            internalErrorServlet.service(request, response);
        }
    }
} 
```
#### 🔍 servletMap
- `key=value` 형식으로 구성되어 있다. URL의 요청 경로가 Key이다.
- **defaultServlet**: `HttpServlet`을 찾지 못할 때 기본으로 실행된다.
- **notFoundErrorServlet**: `PageNotFoundException`이 발생할 때 실행된다.
  - URL 요청 경로를 `servletMap`에서 찾을 수 없고, `defaultServlet`도 없는 경우 `PageNotFoundException`을 던진다.
- **internalErrorServlet**: 처리할 수 없는 예외가 발생하는 경우 실행된다.

```java
public class HttpRequestHandler implements Runnable {
    private final Socket socket;
    private final ServletManager servletManager;
    
    public HttpRequestHandler(Socket socket, ServletManager servletManager) {
        this.socket = socket;
        this.servletManager = servletManager;
    }
    
    @Override
    public void run() {
        try {
            process(socket);
        } catch (Exception e) {
            log(e);
            e.printStackTrace();
        }
    }
    
    private void process(Socket socket) throws IOException {
        try (socket;
             BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream(), UTF_8));
             PrintWriter writer = new PrintWriter(new OutputStreamWriter(socket.getOutputStream(), false, UTF_8))) {
            
            HttpRequest request = new HttpRequest(reader);
            HttpResponse response = new HttpResponse(writer);
            
            servletManager.execute(request, response);
            response.flush();
        }
    }
}
```
- `HttpRequest`, `HttpResponse`를 만들고, `servletManager`에 전달하면 된다.

```java
public class HttpServer {
    
    private final ExecutorService es = Executors.newFixedThreadPool(10);
    private final int port;
    private final ServletManager servletManager;
    
    public HttpServer(int port, ServletManager servletManager) {
        this.port = port;
        this.servletManager = servletManager;
    }
    
    public void start() throws IOException {
        ServerSocket serverSocket = new ServerSocket(port);
        
        while (true) {
            Socket socket = serverSocket.accept();
            es.submit(new HttpRequestHandler(socket, servletManager));
        }
    }
}
```

```java
public class ServerMainV5 {
    
    private static final int PORT = 12345;

  public static void main(String[] args) throws IOException {
    ServletManager servletManager = new ServletManager();
    servletManager.add("/", new HomeServlet());
    servletManager.add("/site1", new Site1Servlet());
    
    HttpServer server = new HttpServer(port, servletManager);
    server.start();
  }
}
```
- 먼저 필요한 서블릿(`HttpServlet`)들을 서블릿 매니저에 등록하자. 이 부분이 바로 서비스 개발을 위한 로직들이다.
- 그리고 `HttpServer`를 생성하면서 서블릿 매니저를 전달하면 된다.

---
---

### 📚 정리
- 이제 HTTP 서버와 서비스 개발을 위한 로직이 명확하게 분리되어 있다.
- 이후에 다른 HTTP 기반의 프로젝트를 시작해야 한다면, HTTP 서버와 관련된 `was.httpserver` 패키지의 코드를 그대로 재사용하면 된다.
  - 그리고 해당 서비스에 필요한 서블릿을 구현하고, 서블릿 매니저에 등록한 다음에 서버를 실행하면 된다.
- 여기서 중요한 부분을 새로운 HTTP 서비스(프로젝트)를 만들어도 `was.httpserver` 부분의 코드를 그대로 재사용할 수 있고, 또 전혀 변경하지 않아도 된다는 점이다.

---

## ✅ 웹 애플리케이션 서버의 역사
- 우리가 만든 `was.httpserver` 패키지를 사용하면 누구나 손쉽게 HTTP 서비스를 개발할 수 있다.
- 복잡한 네트워크, 멀티스레드, HTTP 메시지 파싱에 대한 부분을 모두 여기서 해결해 준다.

#### 🔍 웹 애플리케이션 서버
- 실무 개발자가 목표라면, 웹 애플리케이션 서버(Web Application Server), 줄여서 WAS라는 단어를 많이 듣게 될 것이다.
- Web Server가 아니라 중간에 Application이 들어가는 이유는, 웹 서버의 역할을 하면서 추가로 애플리케이션, 그러니까 프로그램 코드도 수행할 수 있는 서버라는 뜻이다.
- 여기서 말하는 프로그램의 코드는 우리가 작성한 서블릿 구현체들이다.
- 우리가 작성한 서버는 HTTP 요청을 처리하는데, 이때 프로그램의 코드를 실행해서 HTTP 요청을 처리한다.
- 이것이 바로 웹 애플리케이션 서버(WAS)이다.

#### 🔍 서블릿과 웹 애플리케이션 서버
- 초창기에 회사마다 다른 HTTP 서버를 사용하게 되며, 클래스도 다르고 인터페이스도 모두 달라 HTTP 서버를 변경하게 될 경우 코드를 완전히 다 변경해야 하는 문제점이 있었다.
- 이런 문제를 해결하기 위해 1990년대 자바 진영에서는 서블릿(Servlet)이라는 표준이 등장하게 된다.
- 서블릿은 `Servlet`, `HttpServlet`, `ServletRequest`, ServletResponse`를 포함한 많은 표준을 제공한다.

> 서블릿을 제공하는 주요 자바 웹 애플리케이션 서버(WAS)
> - 오픈 소스
>   - Apache Tomcat
>   - Jetty
>   - GlassFish
>   - Undertow
> - 상용
>   - IBM WebSphere
>   - Oracle WebLogic

#### 🔍 참고
- 보통 자바 진영에서 웹 애플리케이션 서버라고 하면 서블릿 기능을 포함하는 서버를 뜻한다.
- 하지만 서블릿 기능을 포함하지 않아도 프로그램 코드를 수행할 수 있다면 웹 애플리케이션 서버라 할 수 있다.

#### 🔍 표준화의 장점
- 성능이나 부가 기능이 더 필요해서 상용 WAS로 변경하거나, 또는 다른 오픈소스 WAS로 변경해도 기능 변경 없이 구현한 서블릿들을 그대로 사용 가능.
- 이것이 바로 표준화의 큰 장점이다.
  - 개발자는 코드의 변경이 거의 없이 다른 애플리케이션 서버를 선택할 수 있다.

---
