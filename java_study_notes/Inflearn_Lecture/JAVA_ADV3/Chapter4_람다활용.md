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

- `GenericFilter`는 제네릭을 사용할 수 있는 모든 타입의 리스트를 람다 조건으로 필터링 할 수 있다.
  - 따라서 매우 유연한 필터링 기능을 제공한다.