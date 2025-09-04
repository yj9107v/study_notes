# 📚 Chapter 13: 애노테이션

> 📌 공부 날짜: 2025/09/04
> - `JAVA `- 고급 2편
> - `References`: [인프런 - 김영한의 실전 자바](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EA%B3%A0%EA%B8%89-2)

---

## ✅ 애노테이션이 필요한 이유
- 만약 리플렉션 같은 기술로 메서드 이름뿐만 아니라 주석까지 읽어서 처리할 수 있다면 좋지 않을까?
  - 그러면 해당 메서드에 있는 주석을 읽어서 URL 경로와 비교하면 된다.
- 그런데 주석은 코드가 아니다. 따라서 컴파일 시점에 모두 제거된다.
- 만약 프로그램 실행 중에 읽어서 사용할 수 있는 주석이 있다면, 그것은 바로 `애노테이션`이다.

#### 🔍 참고: 애노테이션 단어
- `Annotation`은 일반적으로 "주석" 또는 "메모"를 의미한다.
- 애노테이션은 코드에 추가적인 정보를 주석처럼 제공한다.
  - 하지만 일반 주석과 달리, 애노테이션은 컴파일러나 런타임에서 해석될 수 있는 메타데이터를 제공한다.
- 즉, 애노테이션은 코드에 메모를 달아놓은 것처럼 특정 정보나 지시를 추가하는 도구로, 코드에 대한 메타데이터를 표현하는 방법이다.
- 따라서 `애노테이션`이라는 이름은 코드에 대한 추가적인 정보를 주석처럼 달아놓는다는 뜻이다.

---

## ✅ 애노테이션 정의
```java
@Retention(RetentionPolicy.RUNTIME)
public @interface AnnoElement {
    String value();
    int count() default 0;
    String[] tags() default {};
    
    Class<? extends MyLogger> annoData() default MyLogger.class; 
}
```
- 애노테이션은 `@interface` 키워드로 정의한다.
- 애노테이션은 속성을 가질 수 있는데, 인터페이스와 비슷하게 정의한다.

### 📚 애노테이션 정의 규칙
**데이터 타입**
- 기본 타입 (int, float, boolean 등)
- String
- `Class`(메타데이터) 또는 인터페이스
- enum
- 다른 애노테이션 타입
- 위의 타입들의 배열
- 앞서 설명한 타입 외에는 정의할 수 없다. 일반적인 클래스 사용 불가
  - ex) `Member`, `User`, `MyLogger`

**default**
- 요소에 default 값을 지정할 수 있다.
  - ex) `String value() default "기본 값을 적용합니다.";`

**요소 이름**
- 메서드 형태로 정의된다.
- 괄호()를 포함하되, 매개변수는 없어야 한다.

**반환 값**
- `void`를 반환 타입을 사용 불가

**예외**
- 예외를 선언할 수 없다.

**특별한 요소 이름**
- `value`라는 이름의 요소를 하나만 가질 경우, 애노테이션 사용 시 요소 이름을 생략할 수 있다.

---

### 📚 애노테이션 사용
```java
@AnnoElement(value = "data", count = 10, tags = {"t1", "t2"})
public class ElementData1 {}
```
- default 항목은 생략 가능
- 배열의 항목이 하나인 경우 `{}` 생략 가능
- 입력 요소가 하나인 경우 `value` 키워드 생략 가능

```java
public class ElementData1Main {
  public static void main(String[] args) {
    Class<ElementData1> annoClass = ElementData1.class;
    AnnoElement annotation = annoClass.getAnnotation(AnnoElement.class);

    String value = annotation.value();

    int count = annotation.count();

    String[] tags = annotation.tags();
  }
}
```

---

## ✅ 메타 애노테이션
- 애노테이션을 정의하는 데 사용하는 특별한 애노테이션을 메타 애노테이션이라 한다.

**종류**
- `@Retention`
  - `RetentionPolicy.SOURCE`
  - `RetentionPolicy.CLASS`
  - `RetentionPolicy.RUNTIME`
- `@Target`
- `@Documented`
- `@Inherited`

---
#### 🔍 @Retention
애노테이션의 생존 기간을 지정한다.

- `RetentionPolicy.SOURCE`: 소스 코드에만 남아있다. 컴파일 시점에 제거된다.
- `RetentionPolicy.CLASS`: 컴파일 후 class 파일까지는 남아있지만, 자바 실행 시점에 제거된다. (기본 값)
- `RetentionPolicy.RUNTIME`: 자바 실행 중에도 남아있다. 대부분 이 설정을 사용한다.
  - 리플렉션에 사용하려면 반드시 `RUNTIME`

---
#### 🔍 @Target
애노테이션을 적용할 수 있는 위치를 지정한다.

- 이름만으로 충분히 이해가 될 것이다. 주로 `TYPE`, `FIELD`, `METHOD`를 사용한다.

---
#### 🔍 @Documented
자바 API 문서를 만들 때, 해당 애노테이션이 함께 포함되는지 지정한다. 보통 함께 사용한다.

---
#### 🔍 @Inherited
자식 클래스가 애노테이션을 상속받을 수 있다.

---
#### 🔍 적용 예시 
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNPTATION_TYPE)
public @interface Retention {
    RetentionPolicy value();
}
```
- `@Retention`: `RUNTIME` 자바 실행 중에도 애노테이션 정보가 남아있다.
  - 따라서 런타임에 리플렉션을 통해서 읽을 수 있다. 만약 다른 설정을 적용한다면 자바 실행 시점에 애노테이션이 사라지므로 리플렉션을 통해서 읽을 수 없다.
- `@Target`: `ElementType.METHOD, `ElementType.TYPE` 메서드와 타입(클래스, 인터페이스, enum 등)에 `@AnnoMeta` 애노테이션을 적용할 수 있다.
  - 다른 곳에 적용하면 컴파일 오류 발생
- `Documented`: 자바 API 문서를 만들 때, 해당 애노테이션이 포함된다.

---

## ✅ 애노테이션과 상속