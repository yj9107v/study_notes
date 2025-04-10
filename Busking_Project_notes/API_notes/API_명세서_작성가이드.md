# 🧾 API 명세서 작성 방법 정리
> 작성날짜: 2025/04/05
---

## ✅ API 명세서란?

### * API (Application Programming Interface)란?
- 응용 프로그램 간에 통신하는 방법을 정의한 인터페이스.
- 즉, 프론트엔드 앱과 백엔드 서버가 서로 어떻게 대화할지 정해놓은 약속이다.

➡️ API 명세서는 서버와 클라이언트가 통신할 때 사용할 API의 요청/응답 형식, 경로, 규칙 등을 구체적으로 정리한 문서.
→ 요청: 어떤 URL, 어떤 HTTP 메서드, 어떤 JSON 데이터?
→ 응답: 어떤 상태코드, 어떤 JSON 구조로 오는지?
이런 걸 정확하게 명세 해둔 게 바로 API 명세서이다.

---

## ✅ Step1: API 기능 나누기, 나누는 기준은?

### * 나누는 기준
1. 리소스(데이터) : 어떤 대상에 대한 API인가?
   - ex) User, PromotionPost, Review...
2. 동작(행위): CRUD 중심으로 어떤 작업을 하는가?
   - ex) 생성(Create), 조회(Read), 수정(Update), 삭제(Delete)

➡️ 기능은 ERD를 기반으로 생가하면 가장 자연스럽게 나온다.
➡️ 하나의 테이블이 하나의 API 그룹이 되는 경우가 많다.
➡️ 로그인, 로그아웃, 인증 관련은 별도의 Auth 도메인으로 나누기도 함.

---

### * User 기능 나누기

User 테이블에 어떤 기능(API)이 필요한지 먼저 생각하기.

| 기능 이름 | 설명 |
| -----------|-----|
| 회원가입 | 사용자가 가입 |
| 로그인 | 사용자 인증 |
| 내 정보 조회 | 로그인한 사용자의 정보 가져오기 |
| 정보 수정 | 닉네임 등 변경 |
| 회원 탈퇴 | 탈퇴 처리 (Soft delete) |

➡️ 이렇게 하나하나 기능 단위로 쪼개는 게 먼저다.

---

## ✅ Step 2: 하나의 기능을 API로 풀기

### * User - 회원가입 API 명세서 작성 예시

| 항목 | 내용 |
|------|------|
| API 이름 | 회원가입 API |
| 요청 URL | POST /api/users/signup |
| HTTP 메서드 | POST |
| 요청 헤더 | Content-Type: application/json |

**요청 Body**
```json
{
  "username": "busking123",
  "password": "securepassword",
  "nickname": "홍길동"
}
```

| 요청 설명 |
- username: 로그인용 ID (20자 제한)
- password: 비밀번호 (암호화됨)
- nickname: 사용자 별명 (10자 제한)

| 응답 코드 | 201 Created |

**응답 Body**
```json
{
  "id": 1,
  "username": "홍길동",
  "created_at": "2025-04-05T20:00:00"
}
```

| 에러 응답 예시 (400 Bad Request) |
```json
{
  "error": "이미 존재하는 아이디입니다."
}
```

---

## ✅ Step 3: 위와 같은 형식으로 다른 기능도 쭉 정리한다.

---

## ✅ API 명세서 예시의 각 항목이 무엇을 의미하나

### 📌 1. API 이름

| 항목 | 설명 |
|------|------|
| ✅ 의미 | 어떤 기능을 하는 API인지 사람이 알아볼 수 있게 붙인 이름 |
| 🔧 예시 | 회원가입 API, 로그인 API, 게시글 작성 API 등 |
| ❗ 주의 | 실제 개발에 사용되진 않지만 문서화용으로 중요 |

### 📌 2. 요청 URL

| 항목 | 설명 |
|------|------|
| ✅ 의미 | 클라이언트(React 등)가 요청을 보낼 때의 경로(주소) |
| 🔧 예시 | /api/users/signup |
| 🧩 구상 | 도메인 + 경로 → https://example.com/api/users/signup |
| 💡 규칙 | RESTful 하게: 리소스(명사)/동작 순으로 쓰는 게 일반적 |

※ [RESTful하게가 무엇인지](실전_고민&질문_정리/RESTful_API_설계가이드.md)

### 📌 3. HTTP 메서드

| 항목 | 설명 |
|------|------|
| ✅ 의미 | 서버에 어떤 동작을 요청할 것인지 정하는 키워드 |

**🔧 주요 명령어**
- GET: 조회
- POST: 생성
- PUT: 전체 수정
- PATCH: 일부 수정
- DELETE: 삭제

💡 예시: 회원가입 = POST, 로그인 = POST, 내 정보 조회 = GET

### 📌 4. 요청 헤더 (Headers)

| 항목 | 설명 |
|------|------|
| ✅ 의미 | 서버에 요청을 보낼 때 함께 담기는 기본 정보(설정) |

**🔧 자주 쓰는 헤더**

| Key | Value |
|-----|-------|
| Content-Type | application/json |
| Authorization | Bearer <JWT 토큰> |

※ [Bearer <JWT 토큰>이 무엇인지](실전_고민&질문_정리/Bearer_JWT_설명.md)

### 📌 5. 요청 Body

| 항목 | 설명 |
|------|------|
| ✅ 의미 | 서버에 전달할 실제 데이터 내용 (POST, PATCH 등에서 사용) |
| 형식 | 대부분 JSON |

**🧩 예시**
```json
{
  "username": "busking123",
  "password": "securepassword",
  "nickname": "홍길동"
}
```

### 📌 6. 요청 설명

| 항목 | 설명 |
|------|------|
| ✅ 의미 | 요청 Body 안의 각 필드가 무엇을 의미하는지 자세히 설명 |
| 💡 예시 |
- username: 로그인에 사용할 ID (중복 불가, 20자 제한)
- password: 비밀번호 (6자 이상, 암호화됨)
- nickname: 사용자 표시 이름 (10자 제한)

### 📌 7. 응답 코드 (Status Code)

| 항목 | 설명 |
|------|------|
| ✅ 의미 | 서버가 요청을 처리한 결과를 숫자로 알려주는 것 |

**🔧 자주 쓰는 코드**

| 코드 | 의미 |
|------|------|
| 200 OK | 요청 성공 (조회 등) |
| 201 Created | 생성 성공 (회원가입, 글쓰기 등) |
| 400 Bad Request | 잘못된 요청 |
| 401 Unauthorized | 인증 실패 |
| 404 Not Found | 대상 없음 |
| 500 Internal Server Error | 서버 에러 |

### 📌 8. 응답 Body

| 항목 | 설명 |
|------|------|
| ✅ 의미 | 서버가 요청 처리 후 클라이언트에 보내주는 JSON 데이터 |
|🔧 예시 |
```json
{
  "id": 1,
  "username": "busking123",
  "nickname": "홍길동",
  "created_at": "2025-04-01T13:00:00"
}
```
💡 특징:  회원가입/로그인/글쓰기 등은 결과 객체를 보내는 경우 많음 

### 📌 9. 에러 응답 예시

| 항목 | 설명 |
|------|------|
| ✅ 의미 | 요청이 실패했을 때 클라이언트에 보내는 에러 메세지 예시 |

**예시**
```json
{
  "error": "이미 존재하는 아이디입니다."
}
```

💡 팁: 실제 서버에서는 에러 코드, 메시지를 함께 보내도록 구성하는 것이 좋아요

---

## ✅ 요약: API 명세서 한눈에 흐름

**[요청 정보]**
→ 어디에 (URL)  
→ 어떤 방식으로 (메서드)  
→ 무엇을 보낼지 (헤더, 바디)

**[응답 정보]**
← 어떤 결과가 오는지 (응답코드, 바디, 에러)

---

> References
- ChatGPT Plus

