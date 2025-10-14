# 📚 Chapter 4: 람다 활용

> 📌 공부 날짜: 2025/10/13
> - `JAVA `- 고급 3편
> - `References`: [인프런 - 김영한의 실전 자바](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EA%B3%A0%EA%B8%89-3)

---

## ✅ 필터 만들기 1
```java
public class GenericFilter {
    
    public static <T> List<T> filter(List<T> list, Predicate<T> predicate) {
        List<T> result = new ArrayList<>();
        
        for (T n : list) {
            if (predicate.test()) list.add(n);
        }
        
        return result;
    }
}
```

```java
public class FilterMain {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // 짝수만 거르는 필터 (결과: 2, 4, 6, 8, 10)
        List<Integer> evenNumbers = GenericFilter.filter(n -> n % 2 == 0);
        
        // 홀수만 거르는 필터 (결과: 1, 3, 5, 7, 9)
        List<Integer> oddNumbers = GenericFilter.filter(n -> n % 2 = 1);
        
        // 문자 사용 필터 (결과: BB, CCC)
        List<String> strings = List.of("A", "BB", "CCC");
        List<String> stringResult = GenericFilter.filter(strings, s -> s.length() >= 2);
    }
}
```

- `GenericFilter`는 제네릭을 사용할 수 있는 모든 타입의 리스트를 람다 조건으로 필터링할 수 있다.
  - 따라서 매우 유연한 필터링 기능을 제공한다.

---

## ✅ 맵 만들기 1
맵(map)은 대응, 변환을 의미하는 매핑(mapping)의 줄임말이다.

매핑은 어떤 것을 다른 것으로 변환하는 과정을 의미한다.

어떤 하나의 데이터를 다른 데이터로 변환하는 작업.

```java
public class GenericMapper {
    
    public static <T, R> List<R> map(List<T> list, Function<T, R> mapper) {
        List<R> result = new ArrayList<>();
        for (R s : list) {
            result.add(mapper.apply());
        }
        return result;
    }
}
```

```java
public class MapMain {

    public static void main(String[] args) {
        List<String> list = List.of("1", "12", "123", "1234");
        
        // 문자열을 숫자로 변환
        List<Integer> numbers = GenericMapper.map(list, s -> Integer.valueOf(s));
        
        // 문자열의 길이
        List<Integer> lengths = GenericMapper.map(list, s -> s.length());
        
        // *, **, ***
        List<Integer> integers = List.of(1, 2, 3);
        List<String> startList = GenericMapper.map(integers, s -> "*".repeat(s));
    }
}
```
- `String.repeat(int count)`는 자바 11에 추가된 메서드이다. 같은 문자를 `count` 수만큼 붙여서 반환한다.
- `GenericMapper`는 제네릭을 사용할 수 있는 모든 타입의 리스트를 람다 조건으로 변환(매핑)할 수 있다. 따라서 매우 유연한 매핑(변환) 기능을 제공한다.

### 🤔 필터와 맵 활용 1
- 문제 조건: 리스트 값 중에 짝수만 남기고, 남은 짝수 값의 2배를 반환하라.
```java
public class Ex1_number {

    public static void main(String[] args) {
      List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

      List<Integer> directResult = direct(numbers);

      List<Integer> lambdaResult = lambda(numbers);
    }
    
    // 람다를 사용하지 않은 함수
    static List<Integer> direct(List<Integer> numbers) {
        List<Integer> result = new ArrayList<>();
        
        for (Integer i : numbers) {
            if (i % 2 == 0) result.add(i * 2);
        }
        return result;
    }
    
    // 람다를 사용한 함수
    static List<Integer> lambda(List<Integer> numbers) {
        return GenericMapper.map(GenericFilter.filter(i -> i % 2 == 0), i * 2);
    }
}
```

`direct()`와 `lambda()`는 서로 다른 프로그래밍 스타일을 보여준다.

`direct()`는 프로그램을 어떻게 수행해야 하는지 수행 절차를 명시한다.
- 개발자가 로직 하나하나를 어떻게 실행해야 하는지 명시
- 이런 프로그래밍 방식을 명령형 프로그래밍이라 함
- 명령형 스타일은 익숙하고 직관적이나, 로직이 복잡해질수록 반복 코드가 많아질 수 있다.


`lambda()`는 무엇을 수행해야 하는지 원하는 결과에 초점을 맞춘다.
- 특정 조건으로 필터하고, 변환하라고 선언하면, 구체적인 부분은 내부에서 수행된다.
- 개발자는 필터 하고 변환하는 것 즉 무엇을 해야 하는가에 초점을 맞춘다.
  - 예를 들어 시제 어떻게 for문과 if문 등을 사용해서 필터하고 변환할지를 개발자가 크게 신경 쓰지 않는다.
- 이런 프로그래밍 방식을 **선언적 프로그래밍**이라 한다.
- 선언형 스타일은 **무엇**을 하고자 하는지가 명확히 드러난다. 따라서 코드 가독성과 유지보수가 쉬워진다.
  - 여기서는 필터하고 변환하는 것에만 초점을 맞춘다. 실제 어떻게 필터 하고 변환할지는 해당 기능을 사용하는 입장에서는 신경 쓰지 않는다.

---

### 📚 명령형 vs 선언적 프로그래밍

#### 🔍 명령형 프로그래밍 (Imperative Programming)
- **정의**: 프로그램이 **어떻게(How)** 수행되어야 하는지, 즉 **수행 절차**를 명시하는 방식이다.
- **특징**:
  - **단계별 실행**: 프로그램의 각 단계를 명확하게 지정하고 순서대로 실행한다.
  - **상태 변화**: 프로그램의 상태(변수 값 등)가 각 단계별로 어떻게 변환하는지 명시한다,
  - **낮은 추상화**: 내부 구현을 직접 제어해야 하므로 추상화 수준이 낮다.
  - **예시**: 전통적인 for 루프, while 루프 등을 명시적으로 사용하는 방식.
- **장점**: 시스템의 상태와 흐름을 세밀하게 제어할 수 있다.

---
#### 🔍 선언적 프로그래밍 (Declarative Programming)
- **정의**: 프로그램이 **무엇(What)**을 수행해야 하는지, 즉 **원하는 결과**를 명시하는 방식이다.
- **특징**:
  - **문제 해결에 집중**: 어떻게(how) 문제를 해결할 지보다 **무엇**을 원하는지에 초점을 맞춘다.
  - **코드 간결성**: 간결하고 읽기 쉬운 코드를 작성할 수 있다. 
  - **높은 추상화**: 내부 구현을 숨기고 원하는 결과에 집중할 수 있도록 추상화 수준을 높인다.
  - **예시**: `filter`, `map` 등 람다의 고차 함수를 활용, HTML, SQL 등
- **장점**: 코드가 간결하고, 의도가 명확하며, 유지보수가 쉬운 경우가 많다.

---
#### 📌 정리
- **명령형 프로그래밍**은 프로그램이 수행해야 할 각 단계와 처리 과정을 상세하게 기술하여, **어떻게** 결과에 도달할지를 명시
- **선언적 프로그래밍**은 원하는 결과나 상태를 기술하여, 그 결과를 얻기 위한 내부 처리 방식은 추상화되어 있어 개발자가 **무엇**을 원하는지에 집중할 수 있게 한다.
- 특히, **람다**와 같은 도구를 사용하며, 코드를 간결하게 작성하여 선언적 스타일로 문제를 해결할 수 있다.

---

## ✅ 스트림 만들기 1
지금까지는 필터와 맵 기능을 별도의 유틸리티에서 각각 따로 제공했다.

이번에는 앞서 만든 필터와 맵을 함께 편리하게 사용할 수 있도록 하나의 객체에 기능을 통합해 보자.

#### 🔍 스트림 1
데이터들이 흘러가면서 필터 되고, 매핑된다. 데이터가 물 흐르듯이 흘러간다는 느낌을 받았을 것이다.
참고로 흐르는 좁은 시냇물을 영어로 스트림이라 함.

이렇듯 데이터가 흘러가면서 필터도 되고, 매핑도 되는 클래스의 이름을 스트림(`Stream`)이라고 짓자.

```java
public class MyStream<T> {
    
    private List<T> internalList;
    
    private MyStream(List<T> internalList) {
        this.internalList = internalList;
    }
    
    // static factory
    public static <T> MyStream<T> of(List<T> internalList) {
        return new MyStream<>(internalList);
    }
    
    public MyStream<T> filter(Predicate<T> predicate) {
        List<T> filtered = new ArrayList<>();
        for (T element : internalList) {
            if (predicate.test()) filtered.add(element);
        }
        return MyStream.of(filtered);
    }
    
    public <R> MyStream<R> map(function<T, R> mapper) {
        List<R> mapped = new ArrayList<>();
        for (T element : internalList) {
            mapped.add(mapper.apply(element));
        }
        return Mystream.of(mapped);
    }
    
    public List<T> toList() {
        return internalList;
    }
    
    public void forEach(Consumber<T> consumber) {
        for (T element : internalList) {
            consumer.accept(element);
        }
    }
}
```

- 스트림은 내부의 데이터 리스트를 `toList()`로 반환할 수 있다.
- 기존 생성자를 외부에서 사용하지 못하도록 `private`으로 설정
  - 이제 `MyStream`를 생성하려면 `of()` 메서드를 사용해야 한다.
- `forEach` 메서드는 최종 데이터에 사용할 예정이다. 최종 데이터는 반환할 것 없이 요소를 하나하나 소비만 하면 되므로 `Consumer`를 사용했다.

#### 🔍 메서드 체인
우리가 만든 `MyStream`은 `filter`, `map`을 호출할 때 자기 자신의 타입을 반환한다.
따라서 자기 자신의 메서드를 연결해서 호출할 수 있다.

```java
static void methodChain(List<Integer> numbers) {
    List<Integer> result = MyStream.of(numbers)
            .filter(n -> n % 2 == 0)
            .map(n -> n * 2)
            .toList();
    
    // 외부 반복
    for (Integer n : result) {
        System.out.println("age: " + n);
    }
    
    // 내부 반복 추가
    MyStream.of(students)
            .filter(s -> s.getName() >= 80)
            .forEach(name -> System.out.println("name: " + name));
}
```

### 📚 정적 팩토리 메서드 - static factory method
정적 팩토리 메서드는 객체 생성을 담당하는 static 메서드로, 생성자(constructor) 대신 인스턴스를 생성하고 반환하는 역할을 한다.
즉, 일반적인 생성자(Constructor) 대신에 클래스의 인스턴스를 생성하고 초기화하는 로직을 캡슐화하여 제공하는 정적(static) 메서드이다.

- **정적 메서드**: 클래스 레벨에서 호출되며, 인스턴스 생성 없이 접근할 수 있다.
- **객체 반환**: 내부에서 생성한 객체(또는 이미 존재하는 객체)를 반환한다
- **생성자 대체**: 생성자와 달리 메서드 이름을 명시할 수 있어, 생성 과정의 목적이나 특징을 명확하게 표현할 수 있다.
- **유연한 구현**: 객체 생성 과정에서 캐싱, 객체 재활용, 하위 타입 객체 반환 등 다양한 로직을 적용할 수 있다.

참고로 인자들을 받아 간단하게 객체를 생성할 때는 주로 `of(...)`라는 이름을 사용한다.

#### 🔍 참고
정적 팩토리 메서드를 사용하면 생성자에 이름을 부여할 수 있기 때문에 보통 가독성이 더 좋아진다.
하지만 반대로 이야기하면 이름도 부여해야 하고, 준비해야 하는 코드도 더 많다.
객체의 생성이 단순한 경우에는 생성자를 직접 사용하는 것이 단순함의 관점에서 더 나은 선택일 수 있다.

---

### 📚 내부 반복 vs 외부 반복
스트림을 사용하기 전에 일반적인 반복 방식은 `for`, `while` 문과 같은 반복문을 직접 사용해서 데이터를 순회하는 **외부 반복(External Iteration)** 방식이었다.
개발자가 직접 각 요소를 반복하며 처리한다.

스트림에서 제공하는 `forEach()` 메서드로 데이터를 처리하는 방식은 **내부 반복(Internal Iteration)**이라고 부른다.
외부 반복처럼 직접 반복 제어문을 작성하지 않고, 반복 처리를 스트림 내부에 위임하는 방식이다.
**스트림 내부에서** 요소들을 순회하고, 우리는 처리 로직(람다)만 정의해 주면 된다.

반복 제어를 스트림이 대신 수행하므로, 사용자는 반복 로직을 신경 쓸 필요가 없다.
코드가 훨씬 간결해지며, **선언형 프로그래밍 스타일을 적용**할 수 있다.

#### 🔍 내부 반복 vs 외부 반복 선택
많은 경우 내부 반복을 사용할 수 있다면 내부 반복이 선언형 프로그래밍 스타일로 직관적이기 때문에 더 나은 선택이다.
다만 때때로 외부 반복을 선택하는 것이 더 나은 경우도 있다.

1. **외부 반복을 선택하는 것이 더 나은 경우**
- 단순히 한 두 줄 수행만 필요한 경우
- 반복 제어에 대한 복잡하고 세밀한 조정이 필요할 경우

1-1. **단순이 한 두 줄 수행만 필요한 경우**
- 예를 들어, 어떤 리스트를 순화하며 그대로 출력만 하면 되는 등의 매우 간단한 작업을 할 때에는, 굳이 스트림을 사용할 필요가 없다.

1-2. **반복 제어에 대한 복잡하고 세밀한 조정이 필요한 경우**
- 예를 들어 반복 중에 특정 조건을 만나면 바로 반복을 멈추거나, 일부만 건너뛰고 싶다면 `break`, `continue` 등을 사용하는 외부 반복이 단순하다.

#### 📌 정리
연속적인 필터링, 매핑, 집계가 필요할 때는 스트림을 사용한 내부 반복이 선언적이고 직관적이다.
아주 간단한 반복이거나, 중간에 `break`, `continue`가 들어가는 흐름 제어가 필요한 경우는 외부 반복이 더 간결하고 빠르게 이해될 수 있다.

---

## 📌 최종 정리
- **명령형(Imperative) vs 선언형(Declarative) 프로그래밍**
  - **명령형 프로그래밍**은 **어떻게(How)** 문제를 해결할지 로직 단계별로 명령(지시)을 상세히 기술한다.
    - 주로 `for`, `if`와 같은 제어문을 사용하며, 로직이 복잡해질수록 중복 코드가 늘어날 수 있다.
  - **선언형 프로그래밍**은 **무엇(what)**을 해야 하는지에 집중한다. 예를 들어 "짝수만 필터 하고, 그 값을 2배로 변환"처럼 **원하는 결과**만 기술하면, 내부의 세부 로직(어떻게 필터링하고 변환하는지)은 외부에서 신경 쓰지 않는다.
    - 이는 코드 가독성과 유지보수성을 높일 수 있다.

- **Filter와 Map**
  - 조건에 맞는 값만 **선별**하는 작업을 **필터**라고 하고, 값을 다른 값으로 **변환**하는 과정을 **맵(Map)**이라고 한다.

- **Stream (스트림)**
  - 필터와 맵을 포함한 여러 연산을 연속해서 적용하기 위해, 이를 하나의 흐름(스트림)으로 표현한 것이다.
  - 스트림을 사용하면 **메서드 체인 방식**으로 `filter()`, `map()`, `forEach()` 등을 연결해 호출할 수 있으므로, 중간 변수를 만들 필요 없이 깔끔하게 데이터를 가공할 수 있다.
  - **내부 반복(Internal Iteration)**을 지원해, 개발자가 명시적으로 `for` 루프를 작성하지 않고도 반복 처리 로직을 스트림 내부에 위임할 수 있다.

- **정적 팩토리 메서드 (static factory method)**
  - 객체 생성 과정을 메서드로 캡슐화하여 가독성을 높이는 기법. 예) `MyStream.of(list)`
  - 생성자에 이름을 붙이지 못하는 한계를 보완하며, 객체 캐싱 또는 하위 타입 객체 반환 같은 유연한 로직을 적용할 수 있다.

**필터(Filter)**와 **맵(Map)**, 그리고 이를 포괄적으로 사용할 수 있는 **스트림(Stream)**을 적극적으로 활용하면 **선언형 프로그래밍** 스타일로 직관적인 코드를 작성할 수 있다.

반면에 더 세밀한 제어가 필요하다면 **명령형 프로그래밍**을 선택할 수도 있으므로, 상황에 따라 두 방식을 적절히 활용하면 된다.

---
