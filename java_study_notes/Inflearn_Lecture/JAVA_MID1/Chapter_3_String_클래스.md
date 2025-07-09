# 📚 (JAVA 인프런 - 중급 1편) Chapter 3: String 클래스 정리 (2025/03/05)

---

## ✅ 문자열 생성 방법
```java
// 1. 리터럴 방식
String str1 = "hello";

// 2. 생성자 방식
String str2 = new String("hello");
```
- String은 참조형이지만 자바에서 자주 사용하므로 1번과 같이 작성해도 자동으로 처리됨 (문자열 풀 활용)

---

## ✅ String 클래스 내부 구조
- Java 9 이전: `private final char[] value;`
- Java 9 이후: `private final byte[] value;`
  - 내부 구조를 숨기고, 다양한 문자열 처리 기능을 제공
  - 개발자가 불편한 `char[]`을 직접 다루지 않게 함

---

## ✅ char[] 대신 byte[] 사용 이유
- `char`는 2byte, 하지만 영어/숫자는 대부분 1byte면 충분
- 영어/숫자 사용 시 1byte만 사용하여 메모리 절약
- 한글 등은 UTF-16(2byte) 인코딩 사용

---

## ✅ 문자열 연산
- 참조값은 `+` 연산 불가능하지만, String은 `+`, `concat()` 메서드 지원
- 자주 사용되므로 편의성을 위해 지원됨

---

## ✅ 문자열 비교는 `equals()`
- `String` 클래스는 `equals()` 메서드를 **재정의**하여 내부 문자열 비교
- `==`는 참조값 비교 → 같은 문자열이어도 false가 나올 수 있음

### 예시 1: new 키워드 사용
```java
String str1 = new String("hello"); // X001
String str2 = new String("hello"); // X002
System.out.println(str1 == str2); // false
System.out.println(str1.equals(str2)); // true
```

### 예시 2: 리터럴 사용
```java
String str1 = "hello"; // X003
String str2 = "hello"; // X003
System.out.println(str1 == str2); // true
```

- 자바는 문자열 리터럴을 **문자열 풀(String Pool)** 에 저장하여 동일한 문자열 재사용

---

## ✅ 문자열 풀(String Pool)이란?
- 자바 힙 영역에 있는 **공용 문자열 저장소**
- 같은 문자열 리터럴은 한 번만 생성, 여러 변수에서 공유
- 빠른 검색 위해 **해시 알고리즘 사용**

---

## ✅ String은 불변(Immutable) 객체
- 생성 이후 문자열을 **절대 변경 불가**
- 변경 시 기존 문자열은 유지, **새로운 문자열 인스턴스 반환**

### 📌 불변의 이유
- 문자열 풀이 공유되므로 값이 바뀌면 **다른 변수에도 영향**
- 의도치 않은 사이드 이펙트 방지를 위해 불변 설계

---

## ✅ 가변 문자열: StringBuilder
### 📌 String 클래스의 단점
- 변경 시마다 새로운 객체 생성 → 성능 저하, 메모리 낭비

### 📌 StringBuilder 특징
- 내부 배열: `byte[] value;` (mutable)
- 문자열을 직접 변경 가능
- `toString()`으로 불변 String으로 변환 가능

```java
StringBuilder sb = new StringBuilder();
sb.append("ABC");
String result = sb.toString();
```

---

## ✅ StringBuilder 사용이 적합한 상황
1. 반복문에서 문자열을 계속 연결할 때
2. 조건에 따라 문자열을 조립할 때
3. 특정 부분만 변경할 때
4. 대용량 문자열을 다룰 때

---

## ✅ StringBuilder vs StringBuffer
| 항목 | StringBuilder | StringBuffer |
|------|---------------|--------------|
| 쓰레드 안전성 | ❌ (비동기)     | ✅ (동기화됨) |
| 성능           | 빠름           | 느림           |
| 사용 용도       | 단일 쓰레드 환경 | 멀티 쓰레드 환경 |

---

## ✅ 메서드 체이닝 (Method Chaining)
```java
public ValueAdder add(int addValue) {
    value += addValue;
    return this; // 자신의 참조값 반환
}
```
- 메서드 호출을 연속해서 체이닝 가능
- 코드 간결, 가독성 향상

---

## ✅ StringBuilder의 메서드 체이닝 예시
```java
String result = sb.append("A")
                  .append("B")
                  .insert(4, "JAVA")
                  .toString();
```
- 대부분 메서드가 `this` 반환 → 체이닝 가능

---

## ✅ String 최적화 관련 팁
- 리터럴 + 리터럴은 컴파일 시점에 자동 병합됨 → 빠름
- 변수 + 변수는 런타임에 처리됨 → `StringBuilder`로 처리 권장

### 반복문에서 문자열 연결 성능 비교
| 방법 | 시간 |
|------|------|
| `+` 연산 | 약 2.5초 |
| `StringBuilder` | 약 0.003초 |

---

## ✅ 마무리 한 줄 정리
> **불변 객체 String은 안정성, 가변 객체 StringBuilder는 성능! 상황에 맞게 쓰는 것이 중요하다.** 🚀
