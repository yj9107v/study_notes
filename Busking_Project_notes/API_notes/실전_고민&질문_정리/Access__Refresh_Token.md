# 📌 Access Token과 Refresh Token은 무엇인가?

로그인한 사용자의 인증 상태를 유지하기 위한 JWT 기반 인증 시스템의 두 가지 핵심 구성요소이다.

## ✅ 용어 정리
| 토큰 종류 | 뜻 | 주 역할 |
|---------|----|--------|
| Access Token | 접근 토큰 | 로그인한 사용자가 요청할 때, 서버가 `인증됨`을 확인하는 짧은 수명의 토큰 |
| Refresh Token | 재발급 토큰 | Access Token이 만료됐을 때, 새로운 Access Token을 발급받기 위한 토큰 |

---

## ✅ 흐름으로 확인하기
### 🔐 로그인하면:
1. 서버는 두개의 토큰을 발급함:
    - Access Token (짧게 유지 - 보통 15분~1시간)
    - Refresh Token (오래 유지 - 보통 며칠~수 주)

2. 클라이언트는:
    - Access Token → 요청 시 Authorization: Bearer로 헤더에 붙임.
    - Refresh Token → 로컬 저장 (보통 쿠키 또는 secure storage)

---

## ✅ 왜 둘 다 필요할까?
 | 이유 | 설명 |
 |-----|-----|
 | 보안 | Access Token이 탈취되어도 짧은 시간 후 만료됨 |
 | 사용자 편의 | 사용자가 다시 로그인하지 않아도 Refresh Tokend으로 Access Tokend울 재발급 가능 |
 | 세션 유지 | Refresh Token은 로그인 상태를 유지하게 해줌 (백그라운드 자동 재로그인 느낌) |

---

## ✅ 예시 흐름
```text
[로그인 요청]
→ 서버: Access Token + Refresh Token 발급

[요청 시]
→ 헤더에 Access Token 넣어서 요청

[Access Token 만료됨]
→ 서버가 401 Unauthorized 응답

[Refresh Token 요청]
→ 서버: 새로운 Access Token 발급

[Refresh Token도 만료됨]
→ 다시 로그인 필요
```

---

## ✅ 정리 요약
 | 구분 | Access Token       | Refresh Token |
 |-----|--------------------|---------------|
 | 역할 | API 요청 인증용         | Access Token 재발급용 |
 | 유효기간 | 짧음 (15분~1시간)       | 김 (7일~30일) |
 | 저장 위치 | 메모리 / LocalStorage | HttpOnly Cookie 등 |
 | 보안 위험 | 탈취 시 짧은 피해         | 탈취 시 새 Access Token 발급당할 수 있음 |

---

## ✨ 추가 팁
- Refresh Token은 DB에 저장하거나 성능을 고려하면 Redis에 관리해서 발급 시점과 일치하는지 검증하는 방식도 많이 사용.

