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
모든 애노테이션은 `java.lang.annotation.Annotation` 인터페이스를 묵시적으로 상속받는다.

`java.lang.annotation.Annotation` 인터페이스는 개발자가 직접 구현하거나 확장할 수 있는 것이 아니라, 자바 언어 자체에서 애노테이션을 위한 기반으로 사용된다.
이 인터페이스는 다음과 같은 메서드를 제공
- `boolean equals(Object obj)`: 두 애노테이션의 동일성을 비교한다.
- `int hashCode()`: 애노테이션의 해시코드를 반환
- `String toString()`: 애노테이션의 문자열 표현을 반환
- `Class<? extends Annotation> annotationType()`: 애노테이션의 타입을 반환

모든 애노테이션은 기본적으로 `Annotation` 인터페이스를 확장하며, 이로 인해 자바에서 애노테이션은 특별한 형태의 인터페이스로 간주된다.
하지만 자바에서 애노테이션을 정의할 때, 개발자가 명시적으로 `Annotation` 인터페이스를 상속하거나 구현할 필요는 없다. 애노테이션을 `@interface` 키워드를 통해 정의하면, 자바 컴파일러가 자동으로 `Annotation` 인터페이스를 확장하도록 처리해 준다.

#### 🔍 애노테이션 정의
```java
public @interface MyCustomAnnotation {}
```

#### 🔍 애노테이션과 상속
- 애노테이션은 다른 애노테이션이나 인터페이스를 직접 상속할 수 없다.
- 오직 `java.lang.annotation.Annotation` 인터페이스만 상속한다.
- 따라서 애노테이션 사이에는 상속이라는 개념이 존재하지 않는다.

---
### 📚 @Inherited
애노테이션을 정의할 때 `@Inherited` 메타 애노테이션을 붙이면, 애노테이션을 적용한 클래스의 자식도 해당 애노테이션을 부여받을 수 있다.

**단 주의할 점으로 이 기능은 클래스 상속에서만 작동하고, 인터페이스의 구현체에는 적용되지 않는다.**

#### 🔍 @Inherited가 클래스 상속에만 적용되는 이유
**1. 클래스 상속과 인터페이스 구현의 차이**
- 클래스 상속은 자식 클래스가 부모 클래스의 속성과 메서드를 상속받는 개념이다.
  - 즉, 자식 클래스는 부모 클래스의 특성을 이어받으므로, 부모 클래스에 정의된 애노테이션을 자식 클래스가 자동으로 상속받을 수 있는 논리적 기반이 있다.
- 인터페이스는 메서드의 시그니처만을 정의할 뿐, 상태나 행위를 가지지 않기 때문에, 인터페이스의 구현체가 애노테이션을 상속한다는 개념이 잘 맞지 않는다.

**2. 인터페이스와 다중 구현, 다이아몬드 문제**
- 인터페이스는 다중 구현이 가능하다. 만약 인터페이스의 애노테이션을 구현 클래스에서 상속하게 되면 여러 인터페이스의 애노테이션 간의 충돌이나 모호한 상황이 발생할 수 있다.

---

## ✅ 애노테이션 기반 검증기
**참고**
- 자바 진영에서는 애노테이션 기반 검증 기능을 Jakarta(Java) Bean Validation이라는 이름으로 표준화 했다.
- 다양한 검증 애노테이션과 기능이 있고, 스프링 프레임워크, JPA 같은 기술들과도 함께 사용된다.
  - https://beanvalidation.org/
  - https://hibernate.org/validator/

---

## ✅ 자바 기본 애노테이션
`Override`, `@Deprecated`, `@SuppressWarnings`와 같이 자바 언어가 기본으로 제공하는 애노테이션도 있다.
참고로 앞서 설명한 `@Retention`, `@Target`도 자바 언어가 기본으로 제공하는 애노테이션이지만, 이것은 애노테이션 자체를 정의하기 위한 메타 애노테이션이고,
지금 설명할 내용은 코드에 직접 사용하는 애노테이션이다.

#### 🔍 @Override
```java
@Override
public void call() {}
```
- 메서드 재정의가 정확하게 잘 되었는지 컴파일러가 체크하는데 사용한다.
- 개발자의 실수를 자바 컴파일러가 잡아주는 좋은 애노테이션이기 때문에, 사용을 강하게 권장한다.

`@Override`의 `@Retention(RetentionPolicy.SOURCE)` 부분을 보자
- `RetentionPolicy.SOURCE`로 설정하면 컴파일 이후에 `@Override` 애노테이션은 제거된다.
- 컴파일 시점에만 사용하는 애노테이션이여서 런타임에는 필요하지 않으므로 이렇게 설정되어 있다.

---
#### 🔍 @Deprecated
`@Deprecated`는 더 이상 사용되지 않는다는 뜻이다. 이 애노테이션이 적용된 기능은 사용을 권장하지 않는다.

예시)
- 해당 요소를 사용하면 오류를 발생할 가능성이 있다.
- 호환되지 않게 변경되거나 향후 버전에서 제거될 수 있다.
- 더 나은 최신 대체 요소로 대체되었다.
- 더 이상 사용되지 않는 기능이다.

```java
@Deprecated
public void call2() {} // 호출 시 IDE 경고

@Deprecated(since = "2.4", forRemoval = true)
public void call3() {} // 호출 시 IDE 경고(심각)
```
- `@Deprecated`: 더는 사용을 권장하지 않는 요소이다.
  - `since`: 더 이상 사용하지 않게된 버전 정보
  - `forRemoval`: 미래 버전에 코드가 제거될 예정이다.
- `@Deprecated`만 있는 코드를 사용할 경우 IDE에서 경고를 보낸다.
- `forRemoval`이 함께 있는 경우 IDE는 빨간색으로 심각한 경고를 나타낸다.

**`@Deprecated`는 컴파일 시점에 경고를 나타내지만, 프로그램은 작동한다.**

---
#### 🔍 @SuppressWarnings
이름 그대로 경고를 억제하는 애노테이션이다. 자바 컴파일러가 문제를 경고하지만, 개발자가 해당 문제를 잘 알고 있기 때문에,
더는 경고하지 말라고 지시하는 애노테이션이다.

```java
@SuppressWarnings("unused")
public void unusedWarning() {
    int unusedVariable = 10;
}

@SuppressWarnings("deprecation")
public void deprecatedMethod() {
    // 더 이상 사용되지 않는 메서드 호출
    java.util.Date date = new java.utill.Date();
    int date1 = date.getDate();
}

@SuppressWarnings({"rawtypes", "unchecked"})
public void uncheckedCast() {
    // 제네릭 타입 캐스팅 경고 억제,raw type 사용 경고
    List list = new ArrayList();
    
    // 제네릭 타입과 관련된 unchecked 경고
    List<String> stringList = (List<String>) list;
}

@SuppressWarnings("all")
public void suppressAllWarnings() {
    Date date = new Date();
    date.getDate();
    List list = new ArrayList();
    List<String> stringList = (List<String>)list;
}
```
`@SuppressWarnings`에 사용하는 대표적인 값들은 다음과 같다.
- `all`: 모든 경고를 억제
- `deprecation`: 사용이 권장되지 않는(deprecated) 코드를 사용할 때 발생하는 경고를 억제
- `unchecked`: 제네릭 타입과 관련된 unchecked 경고를 억제
- `serial`: Serializable 인터페이스를 구현할 때 serialVersionUID 필드를 선언하지 않는 경우 발생하는 경고를 억제
- `rawtypes`: 제네릭 타입이 명시되지 않은(raw) 타입을 사용할 때 발생하는 경고를 억제
- `unused`: 사용되지 않는 변수, 메서드, 필드 등을 선언했을 때 발생하는 경고를 억제

---

## 📌 정리
이러한 프레임워크들은 리플렉션과 애노테이션을 활용하여 다음의 "마법 같은" 기능들을 제공한다.

**1. 의존성 주입(Dependency Injection)**: 스프링은 리플렉션을 사용하여 객체의 필드나 생성자에 자동으로 의존성을 주입한다.
개발자는 단순히 `@Autowired` 애노테이션만 붙이면 된다.

**2. ORM(Object-Relational Mapping)**: JPA는 애노테이션을 사용하여 자바 객체와 데이터베이스 테이블 간의 매핑을 정의한다.
예를 들어, `@Entity`, `@Table`, `@Column` 등의 애노테이션으로 객체-테이블 관계를 설정한다.

**3. AOP(Aspect-Oriented Programming)**: 스프링은 리플렉션을 사용하여 런타임에 코드를 동적으로 주입하고,
`@Aspect`, `@Before`, `@After` 등의 애노테이션으로 관점 지향 프로그래밍을 구현한다.

**4. 설정의 자동화**: `@Configuration`, `@Bean` 등의 애노테이션을 사용하여 다양한 설정을 편리하게 적용한다.

**5. 트랜잭션 관리**: `@Transactional` 애노테이션만으로 메서드 레벨의 DB 트랜잭션 처리가 가능해진다.

- 이러한 기능들은 개발자가 비즈니스 로직에 집중할 수 있게 해주며, 보일러플레이트(지루한 반복) 코드를 크게 줄여준다.
- 프레임워크 동작 원리를 깊이 이해하기 위해서는 리플렉션과 애노테이션에 대한 이해가 필수다.
  - 이를 통해 프레임워크가 제공하는 편의성과 그 이면의 복잡성 사이의 균형을 잡을 수 있으며, 필요에 따라 프레임워크를 효과적으로 커스마이징하거나 최적화할 수 있게 된다.

스프링이나 JPA 같은 프레임워크들은 리플렉션과 애노테이션을 극대화해서 사용한다.

---
