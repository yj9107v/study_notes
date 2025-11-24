# 📖 지식 정리 (우테코를 하며 학습한 내용)
## 1️⃣ Stream 함수 공부

### stream()
- 리스트를 스트림 형태로 변환하여 하나씩 순회 가능한 형태로 만듦

### distinct()
- 스트림에서 중복되는 값을 제거함.

```java
// 활용 예시
private final List<Integer> numbers;

// ... 로또 번호에서 중복된 숫자가 있는지 검증
if (numbers.stream().distinct().count() != 6) {}
```
로또 번호 리스트를 하나씩 순회하여 중복이 있는지 검증하고, 중복되는 값은 제거하여 개수를 파악.
count 값이 6개여야 중복된 값이 없는 것, 6개가 아닌 경우 중복된 값이 있으므로 예외 처리!

---

### Stream.generate(Supplier<T> s)
- `java.util.stream.Stream`의 정적 메서드
- 인자로 `Supplier<T>`를 받아서, 해당 `Supplier`를 계속 호출해서 **무한 스트림(infinite stream)**을 만들어냄

#### .limit(long maxSize)
- 원래 무한 스트림이었던 것을 **앞에서 부터 최대 `maxSize`개까지만 사용**하도록 잘라내는 연산이다.

```java
public List<Lotto> issueTickets(int ticketCount) {
    return Stream.generate(this::issue)
            .limit(ticketCount)
            .toList();
}
```

### 함께 나누기 정보
함께 나누기에서 나와 같이 Stream 활용에 고민하시던 분이 있었고, 그 분이 추천해준 Stream API 연습 문제 사이트를 공유받았다.
이번 우테코가 끝나서도 Stream 활용이 헷갈린다면 이 API를 통해 더 학습하려고 한다.

> [Java Stream API 연습 문제 100선](https://github.com/gywns0417/java-stream-practice)


---

## ☑️ 고민한 내용
> - 그동안 `stream` 문법이 익숙하지 않아 `for문`으로만 작성하고,
    > stream을 사용해도 AI의 도움 수준에서만 이해하는 경우가 많았다.
> - 이번 오픈 미션에서는 **직접 사용해 보고, 필요한 함수들을 찾아보며 `stream`을 내 것으로 만드는 것**에 집중했다.
> - 예를 들어 Lotto 번호 중복 검증은 `numbers.stream().distinct().count() != 6`를 사용하여,
    > 중복 제거(distinct) → 요소 개수(count)로 **중복 여부를 깔끔하게 판단할 수 있었다**.
    > 단순 문법 활용이 아니라 “왜 이 방식이 더 좋은가?”까지 생각하게 되면서 `stream` 사용의 의도와 장점을 제대로 이해하게 되었다.
> - 즉, 이번 미션을 통해 `stream`을 “그냥 쓰는 문법”이 아니라 **내가 선택하고 설계하는 도구**로 사용할 수 있게 되었다.

---

## 🔖 References
