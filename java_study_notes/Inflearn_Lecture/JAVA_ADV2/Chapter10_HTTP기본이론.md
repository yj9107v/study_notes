# 📚 Chapter 10: HTTP - 기본 이론

> 📌 공부 날짜: 2025/08/14
> - `JAVA `- 고급 2편
> - `References`: 인프런 - 김영한의 실전 자바

---

## ✅ 클라이언트 서버 구조
- Request Response 구조
- 클라이언트는 서버에 요청을 보내고, 응답을 대기
- 서버가 요청에 대한 결과를 만들어서 응답

---

## ✅ 모든 것이 HTTP
### HTTP 메시지에 모든 것을 전송
- HTML, TEXT
- IMAGE, 음성, 영상, 파일
- JSON, XML(API)
- 거의 모든 형태의 데이터 전송 가능
- 서버 간에 데이터를 주고받을 때도 대부분 HTTP 사용

---

## ✅ HTTP 요청, 응답 메시지
### 📚 요청 데이터 생성 및 전달
`ex) HTTP 요청 메시지`
```text
GET /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com
```

### 📚 응답 데이터 생성 및 전달
`ex) HTTP 응답 메시지`
```text
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF=8
Content-Length: 3423

<html>
    <body>...</body>
</html>
```

---

## ✅ HTTP 메시지 구조
```text
start-line 시작라인
---
header 헤더
---
empty line 공백 라인 (CRLF)
---
message body
---
```

### 📚 시작 라인
#### 🔍 요청 메시지 - HTTP 메서드
- 종류: GET, POST, PUT, PATCH, DELETE...
- 서버가 수행해야 할 동작 지정
  - GET: 리소스 조회
  - POST: 요청 내역 처리(게시물 등록, 계좌 이체)

#### 🔍 요청 메시지 - 요청 대상
- `absolute-path[?query] (절대 경로[?쿼리])`
- `절대 경로 = "/"`로 시작하는 경로
- `쿼리: key1=value1&key2=value2` 형식으로 데이터 전달
- ex) `/search?q=hello&hl=ko`

#### 🔍 요청 메시지 - HTTP 버전
- HTTP Version
- ex) `HTTP/1.1`

#### 🔍 응답 메시지
- HTTP 버전
- HTTP 상태 코드: 요청 성공, 실패를 나타냄
  - 200: 성공
  - 400: 클라이언트 요청 오류, 404 요청 리소스가 없음
  - 500: 서버 내부 오류
- 이유 문구: 사람이 이해할 수 있는 짧은 상태 코드 설명 글
- ex) `HTTP/1.1 200 OK`

### 📚 HTTP 헤더
- HTTP 전송에 필요한 모든 부가정보
- name: value (필드 이름은 대소문자 구문 없음)
- 예) 메시지 바디의 내용, 메시지 바디의 크기, 압축, 인증, 요청 클라이언트(브라우저) 정보, 서버 애플리케이션 정보, 캐시 관리 정보...
- 표준 헤더가 너무 많다
- 필요시 임의의 헤더 추가 가능
- ex) `Host: www.google.com` // 요청
- ex) `Content-Type: text/html;charset=UTF=8` // 응답
- ex) `Content-Length: 3423` // 응답

### 📚 HTTP 메시지 바디
- 실제 전송할 데이터
- HTML 문서, 이미지, 영상, JSON 등등 byte로 표현할 수 있는 모든 데이터 전송 가능

---

## ✅ HTTP 메서드
#### 🔍 HTTP 메서드 종류
- GET: 리소스 조회 (검색 기능)
- POST: 요청 데이터 처리, 주로 등록에 사용 (회원가입 처리, 결제 처리, 데이터 등록 등)
- PUT, PATCH, DELETE...

### 📚 GET
- 리소스 조회
- 서버에 전달하고 싶은 데이터는 query(쿼리 파라미터, 쿼리 스트링)를 통해서 전달
- 메시지 바디는 사용하지 않는다.
- (메시지 바디를 사용해서 데이터를 전달할 수 있지만, 지원하지 않는 곳이 많아서 권장하지 않음)

### 📚 POST
- 요청 데이터 처리
- **메시지 바디를 통해 서버로 요청 데이터 전달**
- 서버는 요청 데이터를 처리
  - 메시지 바디를 통해 들어온 데이터를 처리하는 모든 기능을 수행한다.
- 주로 전달된 데이터로 신규 리소스 등록, 프로세스 처리에 사용

---

## 📌 HTTP는 핵심이기에 공부를 더 자세히 해야 한다.

---