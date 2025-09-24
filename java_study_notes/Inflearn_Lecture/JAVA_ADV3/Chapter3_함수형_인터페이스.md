# 📚 Chapter 3: 함수형 인터페이스

> 📌 공부 날짜: 2025/09/24
> - `JAVA `- 고급 3편
> - `References`: [인프런 - 김영한의 실전 자바](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EA%B3%A0%EA%B8%89-3)

---

## ✅ 함수형 인터페이스와 제네릭
함수형 인터페이스도 인터페이스이기 때문에 제네릭을 도입할 수 있다.

지금까지 개발한 프로그램은 코드 재사용과 타입 안정성이라는 2마리 토끼를 한 번에 잡을 수 없었다.
코드 재사용을 늘리기 위해 `Object`와 다형성을 사용하면 타입 안정성이 떨어지는 문제가 발생한다.
- `StringFunction`, `NumberFunction` 각각의 타입별로 함수형 인터페이스를 정의
  - 코드 재사용 ❌
  - 타입 안정성 🆗
- `ObjectFunction`을 사용해서 Object의 다형성을 활용해서 하나의 함수형 인터페이스만 정의
  - 코드 재사용 🆗
  - 타입 안정성 ❌

함수형 인터페이스에 **제네릭을 도입해서** 코드 재사용도 늘리고, 타입 안정성까지 높일 수 있다.

```java
public class GenericMain {
    public static void main(String[] args) {
        GenericFunction<Integer, Integer> square = n -> n * n;
        GenericFunction<String, String> upperCase = s -> s.toUpperCase();
    }
    
    @FunctionalInterface
    interface GenericFunction<T, R> {
        R apply(T s);
    }
}
```
- `T`: 매개변수의 타입
- `R`: 반환 타입

함수형 인터페이스에 제네릭을 도입한 덕분에 메서드 `apply()`의 매개변수와 반환 타입을 유연하게 변경할 수 있다.

#### 📌 정리
- 제네릭을 사용하면 동일한 구조의 함수형 인터페이스를 다양한 타입에 재사용할 수 있다.
- 컴파일 시점에 타입 체크가 이루어지므로 런타임 에러를 방지할 수 있다.
- 코드의 중복을 줄이고, 유지보수성을 높이는데 큰 도움이 된다.

---

## ✅ 람다와 타깃 타입
### 🤔 남은 문제
우리가 만든 `GenericFunction`은 코드 중복을 줄이고, 유지보수성을 높여주지만 2가지 문제가 있다.

#### 문제 1. 모든 개발자들이 비슷한 함수형 인터페이스를 개발해야 한다.

#### 문제 2. 개발자 A가 만든 함수형 인터페이스와 개발자 B가 만든 함수형 인터페이스는 서로 호환되지 않는다.

문제의 코드
```java
public class TargetType {
    public static void main(String[] args) {
        FunctionA functionA = i -> "value = " + i;
        FunctionB functionB = i -> "value = " + i;
        
        // 이미 만들어진 FunctionA 인스턴스를 FunctionB에 대입: 불가능
        FunctionB targetB = functionA; // 컴파일 에러! ❌
    }
}
```
두 인터페이스 모두 `Integer`를 받아 `String`을 리턴하는 동일한 `apply(...)` 메서드를 갖고 있지만, **자바 타입 시스템상 전혀 다른 인터페이스**이므로 서로 호환되지 않는다.

#### 📌 정리
- **람다**는 익명 함수로서 특정 타입을 가지지 않고, 대입되는 참조 변수가 어떤 함수형 인터페이스를 가리키느냐에 따라 타입이 결정된다.
- 한편 **이미 대입된 변수**(`functionA`)는 엄연히 `FunctionA` 타입의 객체가 되었으므로, 이를 `FunctionB` 참조 변수에 그대로 대입할 수는 없다.
  - 두 인터페이스 이름이 다르기 때문에 자바 컴파일러는 다른 타입으로 간주한다.
- 따라서 시그니처가 똑같은 함수형 인터페이스라도, 타입이 다르면 상호 대입이 되지 않는 것이 자바의 타입 시스템 규칙이다.

---

### 📚 자바가 기본으로 제공하는 함수형 인터페이스
자바는 이런 문제들을 해결하기 위해 필요한 함수형 인터페이스 대부분을 기본으로 제공한다.

자바가 제공하는 함수형 인터페이스를 사용하면, 비슷한 함수형 인터페이스를 불필요하게 만드는 문제는 물론이고, 함수형 인터페이스의 호환성 문제까지 해결할 수 있다.
- 자바는 `java.util.function` 패키지에 다양한 기본 함수형 인터페이스들을 제공한다.

```java
public class TargetType {
    public static void main(String[] args) {
        Function<String, String> upperCase = s -> s.toUpperCase();
        upperCase.apply("hello");
        
        Function<Integer, Integer> square = n -> n * n;
        square.apply(3);
        
        Function<Integer, Integer> square2 = square;
    }
}
```
**따라서 자바 기본으로 제공하는 함수형 인터페이스를 사용하자**

---

## ✅ 기본 함수형 인터페이스
#### 🔍 자바가 제공하는 대표적인 함수형 인터페이스
- `Function`: 입력O, 반환O
- `Consumer`: 입력O, 반환X
- `Supplier`: 입력X, 반환O
- `Runnable`: 입력X, 반환X

함수형 인터페이스들은 대부분 제네릭을 활용하므로 종류가 많을 필요는 없다.
(`Runnable`은 `java.lang` 패키지에 위치, 대부분 `java.util.function`)

---
### 📚 Function
```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```
- 하나의 매개변수를 받고, 결과를 반환하는 함수형 인터페이스이다. (둘 이상의 매개변수를 받는 함수형 인터페이스는 `BiFunction`)
- 입력값(`T`)을 받아서 다른 타입의 출력값(`R`)을 반환하는 연산을 표현할 때 사용한다.
  - 물론 같은 타입의 출력 값도 가능하다. (하지만 보통 `UnaryOperator`를 사용, 이유는 뒤에서 설명)
- 일반적인 함수(Function)의 개념에 가장 가깝다.

```java
Function<String, Integer> function1 = s -> s.length();
function1.apply("hello)"; // 결과: 5
```

---
### 📚 Consumer
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```
- 입력 값(T)만 받고, 결과를 반환하지 않는(`void`) 연산을 수행하는 함수형 인터페이스이다.
- 입력받은 데이터를 기반으로 내부적으로 처리만 하는 경우에 유용하다.
  - 예) 컬렉션에 값 추가, 콘솔 출력, 로그 작성, DB 저장 등

#### 🔍 용어 설명
- Consumer는 "소비자"라는 의미로, 데이터를 받아서 소비(사용)만 하고 아무것도 돌려주지 않는다는 개념을 표현한다.
- accept는 "받아들이다"라는 의미로, 입력 값을 받아들여서 처리한다는 동작을 설명.

```java
Consumer<String> consumer1 = s -> System.out.println(s);
consumer1.accept("hello");
```

---
### 📚 Supplier
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```
- 입력을 받지 않고(`()`) 어떤 데이터를 공급(supply)해주는 함수형 인터페이스이다.
- 객체나 값 생성, 지연 초기화 등에 주로 사용

```java
Supplier<Integer> supplier1 = () -> new Random().nextInt(10);
System.out.println(supplier1.get()); // 결과: 1 ~ 10 랜덤 값이 나온다.
```

---
### 📚 Runnable
```java
@FunctionalInterface
public interface Runnable {
    void run();
}
```
- 입력값도 반환값도 없는 함수형 인터페이스이다.
  - 자바에서는 원래부터 스레드 실행을 위한 인터페이스로 쓰였지만, 자바 8 이후에는 람다식으로도 많이 표현된다.
- `java.lang` 패키지에 있다.
- 주로 멀티스레딩에서 스레드의 작업을 정의할 때 사용

```java
Runnable runnable1 = () -> System.out.println("Hello Runnable");
runnable1.run();
```

### 📌 요약
| 인터페이스 | 메서드 시그니처 | 입력 | 출력                     | 대표 사용 예시        |
|----------|--------------|-----|------------------------|-----------------|
| **Function<T, R>** | R apply(T t) | 1개 (T) | 1개 (R) | 데이터 변환, 필드 추출 등 |
| **Consumer<T>** | void accept(T t) | 1개 (T) | 없음 | 로그 출력, DB 저장 등 |
| **Supplier<T>** | T get() | 없음 | 1개 (T) | 객체 생성, 값 반환 등 |
| **Runnable** | void run() | 없음 | 없음 | 스레드 실행(멀티 스레드) |

---

## ✅ 특화 함수형 인터페이스
특화 함수형 인터페이스는 의도를 명확하게 만든 조금 특별한 함수형 인터페이스다.
- `Predicate`: 입력O, 반환 `boolean`
  - 조건 검사, 필터링 용도
- `Operator(UnaryOperator,BinaryOperator)`: 입력O, 반환 O
  - 동일한 타입의 연산 수행, 입력과 같은 타입을 반환하는 연산 용도

### 📚 Predicate
```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```
- 입력 값(T)을 받아서 `true` 또는 `false`로 구분(판단)하는 함수형 인터페이스이다.
- 조건 검사, 필터링 등의 용도로 많이 사용된다. (스트림 API에서 필터 조건을 지정할 때 자주 등장)

```java
Predicate<Integer> predicate = value -> value % 2 == 0;
predicate.test(2) // 결과 true;
```

#### 🤔 Predicate가 꼭 필요할까?
`Predicate`는 입력이 T, 반환이 `boolean`이기 때문에 결과적으로 `Function<T, boolean>`으로 대체할 수 있다.
그럼에도 불구하고 `Predicate`를 별도로 만든 이유는 다음과 같다.

➡️ `Predicate<T>`는 "입력 값을 받아 `true/false`로 결과를 판단한다"라는 **의도를 명시적으로 드러내기 위해 정의** 된 함수형 인터페이스이다.

별도로 둠으로써 다음과 같은 이점들을 얻을 수 있다.
1. **의미의 명확성**
  - `Predicate<T>`를  사용하면 "이 함수는 조건을 검사하거나 필터링 용도로 쓰인다"라는 **의도가 더 분명**해진다.

2. **가독성 및 유지보수성**
  - 여러 사람과 협업하는 프로젝트에서, "조간을 판단하는 함수"는 `Predicate<T>`라는 패턴을 사용함으로써 전달이 명확해진다.
  - `boolean` 판단 로직이 들어가는 부분에서 `Predicate<T>`를 사용하면 코드 가족성과 유지보수성이 향상된다.
    - 이름도 명시적이고, 제네릭에 `<Boolean>`을 적지 않아도 된다.

정리하면 `Function<T, Boolean>`로도 같은 기능을 구현할 수는 있지만, **목적(조건 검사)과 용도(필터링 등)에 대해 더 분명히 표현하고, 가동성과 유지보수를 위해** `Predicate<T>`라는 별도의 함수형 인터페이스가 마련되었다.

#### 🔍 의도가 가장 중요한 핵심
자바가 제공하는 다양한 함수형 인터페이스들을 선택할 때는 단순히 입력값, 반환값만 보고 선택하는 게 아니라 해당 함수형 인터페이스가 제공하는 **의도**가 중요하다.

---
### 📚 Operator
Operator는 `UnaryOperator`, `BinaryOperator` 2가지 종류가 제공된다.

#### 🔍 용어 설명
- Operator라는 이름은 수학적인 연산자(Operator)의 개념에서 왔다.
- 수학에서 연산자는 보통 **같은 타입의 값들을 받아서 동일한 타입의 결과를 반환한다.**
- 자바에서는 수학처럼 숫자의 연산에만 사용된다기보다는 입력과 반환이 동일한 타입의 연산에 사용할 수 있다.

---
### 📚 UnaryOperator (단항 연산)
```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    T apply(T t);
}
```
- 단항 연산은 **하나의 피연산자(operand)**에 대해 연산을 수행하는 것을 말한다.
  - 예) 숫자의 부호 연산(`-x`), 논리 부정 연산(`!x`) 등
- 입력(피연산자)의 결과(연산 결과)가 **동일한 타입**인 연산을 수행할 때 사용
- `Function<T, T>`를 상속받는데, 입력과 반환을 모두 같은 `T`로 고정한다. 따라서 `UnaryOperator`는 입력과 반환 타입이 반드시 같아야 한다.

---
### 📚 BinaryOperator (이항 연산)
```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    T apply (T t1, T t2);
}
```
- 이항 연산은 **두 개의 피연산자(operand)**에 대해 연산을 수행하는 것을 말한다.
- **같은 타입**의 두 입력을 받아, **같은 타입**의 결과를 반환할 때 사용된다.
- `BiFunction<T, T, T>`를 상속받는 방식으로 구현되어 있는데, 입력값 2개와 반환을 모두 같은 T로 고정한다.
  - 따라서 `BinaryOperator`는 모든 입력값과 반환 타입이 반드시 같아야 한다.
  - `BiFunction`은 입력 매개변수가 2개인 `Function`이다.

```java
public class OperatorMain {
    public static void main(String[] args) {
        Function<Integer, Integer> square1 = x -> x * x;
        UnaryOperator<Integer> square2 = x -> x * x;
        
        BiFunction<Integer, Integer, Integer> addition1 = (a, b) -> a + b;
        BinaryOperator<Integer> addition2 = (a, b) -> a + b;
    }
}
```

#### 📌 정리
**Operator를 제공하는 이유**는 의도의 명시성과 가독성과 유지보수성 향상을 위해 별도로 사용한다.

---

## ✅ 기타 함수형 인터페이스
### 📚 입력 값이 2개 이상
매개변수가 2개 이상 필요한 경우에는 `BiXxx` 시리즈를 사용하면 된다. Bi는 Binary(이항, 둘)의 줄임말이다.
- 예) `BiFunction`, `BiConsumer`, `BiPredicate`

#### 🤔 입력 값이 3개라면?
입력 값이 3개라면 `TriXxx`가 있으면 좋겠지만, 이런 함수형 인터페이스는 기본으로 제공하지 않는다.
보통 잘 사용하지 않기 때문, 3개일 경우라면 직접 만들어서 사용.

---
### 📚 기본형 지원 함수형 인터페이스
다음과 같이 기본형(primitive type)을 지원하는 함수형 인터페이스도 있다.
```java
@FunctionalInterface
public interface IntFunction<R> {
    R apply (int value);
}
```

#### 🔍 기본형 지원 함수형 인터페이스가 존재하는 이유
- 오토박싱/언박싱(auto-boxing/unboxing)으로 인한 성능 비용을 줄이기 위해
- 자바 제네릭의 한계(제네릭은 primitive 타입을 직접 다룰 수 없음)를 극복하기 위해

#### 🔍 Function
- `IntFunction`은 매개변수가 `int`이다.
- `ToIntFunction`은 반환 타입이 기본형 `int`이다.
- `IntToLongFunction`은 매개변수가 `int`, 반환 타입이 `long`이다.
  - `IntToIntFunction`은 없는데, `IntOperator`를 사용하면 된다. (타입이 똑같으니까)

---
#### 📌 정리
**특화 함수형 인터페이스**

| 인터페이스                 | 메서드 시그니처            | 입력        | 출력             | 대표 사용 예시                         |
|-----------------------|---------------------|-----------|----------------|----------------------------------|
| **Predicate<T>**      | boolean test(T t)   | 1개 (T)    | boolean        | 조건 검사, 필터링                       |
| **UnaryOperator<T>**  | T apply(T t)        | 1개 (T)    | 1개 (T; 입력과 동일) | 단항 연산 (예: 문자열 변환, 단항 계산)         |
| **BinaryOperator<T>** | T apply(T t1, T t2) | 2개 (T, T) | 1개 (T; 입력과 동일) | 이항 연산 (예: 두 수의 합, 최댓값 반환) |

- 자바가 기본으로 지원하지 않는다면 직접 만들어서 사용하자.

---

