# 📚 (JAVA 인프런 - 중급 1편) Chapter 4: 래퍼, Class 클래스 정리 (2025/03/06)

---

## ✅ 기본형의 한계

### 1. 객체가 아님
- int, double 같은 기본형은 객체가 아님
- 유용한 메서드 제공 불가
- 컬렉션, 제네릭 등 객체 참조가 필요한 곳에서 사용 불가

### 2. null 값을 가질 수 없음
- 데이터 없음 상태를 표현할 수 없음

---

## ✅ 래퍼 클래스 (Wrapper Class)

- 기본형을 감싸는 클래스
- null 사용 가능 (주의: NullPointerException 발생 가능성 있음)

---

## ✅ 자바의 기본형 ↔ 래퍼 클래스 매핑

| 기본형  | 래퍼 클래스 |
|--------|-------------|
| byte   | Byte        |
| short  | Short       |
| int    | Integer     |
| long   | Long        |
| float  | Float       |
| double | Double      |
| boolean| Boolean     |

- 모든 래퍼 클래스는 불변(immutable)
- 비교는 `equals()` 사용

---

## ✅ 박싱(Boxing)과 언박싱(Unboxing)

### 박싱 (Boxing)
```java
// 사용 지양 (Deprecated 예정)
Integer i1 = new Integer(10);

// 권장 방식
Integer i2 = Integer.valueOf(10);
```

- valueOf()는 -128~127 범위 Integer를 캐싱 (성능 최적화)

### 언박싱 (Unboxing)
```java
int value = i2.intValue();
```

- toString() 메서드 오버라이딩 → 내부 값 출력

---

## ✅ 오토 박싱 / 오토 언박싱

자바에서 자동으로 박싱, 언박싱 수행
```java
Integer num = 10;       // 오토 박싱
int value = num + 1;    // 오토 언박싱
```

---

## ✅ parseInt() vs valueOf()

| 메서드           | 반환 타입 |
|------------------|-----------|
| parseInt("10")   | int       |
| valueOf("10")    | Integer   |

---

## ✅ 성능과 유지보수

- 기본형 연산이 래퍼 클래스보다 몇 배 빠름
- 수만 ~ 수십만 반복 연산 시에만 성능 차이 고려
- 일반적 상황에서는 유지보수성을 우선

> 웹 앱에서는 네트워크 호출이 자바 내부 연산보다 수십만 배 느림  
> → 성능 최적화보다 구조 설계와 네트워크 최적화가 우선!

---

## ✅ Class 클래스

- 클래스의 메타데이터(정보)를 다룸
- 실행 중 클래스의 정보 조회 및 조작 가능

### 주요 기능
1. 타입 정보 조회
2. 리플렉션 (필드, 메서드, 생성자 조회 및 실행)
3. 동적 로딩 및 객체 생성
4. 애노테이션 처리

> Class는 예약어 → 변수명 등에서는 Clazz 사용 관행

---

## ✅ Class 객체 조회 방법

```java
Class clazz1 = String.class;
Class clazz2 = new String().getClass();
Class clazz3 = Class.forName("java.lang.String");
```

---

## ✅ System 클래스

- `System.exit(0)` 사용은 지양 (프로그램 강제 종료)

---

## ✅ Random 클래스

- java.util 패키지 소속
- 다양한 랜덤 값 생성 가능

```java
Random random = new Random();
int value = random.nextInt(10); // 0 ~ 9
int value2 = random.nextInt(10) + 1; // 1 ~ 10
```

### 씨드(Seed)

- Random 객체에 같은 seed 값 설정 시 항상 같은 랜덤 결과 생성
```java
Random fixedRandom = new Random(1); // 동일한 결과
```

- 테스트 코드에서 유용함

---

## ✅ 마무리 한 줄 정리

> **성능보다 유지보수! 래퍼 클래스는 유연하고 편리하다, 최적화는 정말 필요할 때만!** 🔧
