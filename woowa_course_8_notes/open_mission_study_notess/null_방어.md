# 📖 지식 정리 (우테코를 하며 학습한 내용)
## 1️⃣ null
null은 개발자들에게 가장 자신을 괴롭히는 것으로 손에 꼽힌다 한다.

> **NPE: NullPointerException**
> - 이 예외는 `null` 참조자가 할당된 객체의 필드나 메서드에 접근하려고 하면 발생한다.
> - 자바는 `Null Safety`하지 않은 언어이기 때문에, 언제 어디서 `null`이 발생할지 예측하기 힘들다.

따라서 자바 개발자는 항상 `null`이 들어올 것을 대비해 방어적으로 프로그래밍 해야만 한다.

## 2️⃣ NPE를 줄이는 방법 

### 2️⃣-1️⃣ null을 매개변수로 넘기지 말 것
가장 기본적인 것이다. 대부분의 메서드들은 인수가 `null`일 것을 고려하고 설계하지 않는다.

null을 사용해서 조건을 표현하려 하지 말고, 대신 "의도에 맞는 메서드"를 오버로딩하거나 기본값을 설정하는 방식이 낫다.

null은 설계를 복잡하게 만들고, 코드 읽는 사람도 헷갈리게 만든다.

### 2️⃣-2️⃣ 반환값으로 null을 반환하지 말 것
반환값이 존재하는 메서드 중에는 간혹가다 마지막에 `null`이 반환되는 것들이 있다.
이것은 반환할 변수에 마땅히 대입될만한 값이 없어, 처음 초기화 시 대입된 `null` 참조자가 그대로 반환되는 경우가 많다.

그런데 개발자 입장에서 메서드가 `null`을 반환하는 경우는 그다지 달가운 상황이 아니다.
그 이유는 **메인 로직에 `null` 값이 아님을 검증해야 하는 작업을 추가로 작성해야 하는 번거로움이 발생하기 때문이다.**

또한 책임 측면에서 볼 때, 하나의 메서드에서 발생한 책임을 다른 곳으로 떠넘기는 것은 전형적인 안티 패턴이다.
이런 경우에는 `null` 대신 빈 객체를 반환하는 것이 좋다.

만약 일반적인 참조형 변수라서 빈 값을 생성할 수 없다면, 별도의 두 가지 선택지가 존재한다.
하나는 예외를 던지는 것이고, 다른 하나는 `Optional.empty()`를 반환하는 것이다.
`Java8` 이상을 쓰고 있다면 후자를 권장한다.

`Optional`을 사용하는 것은 한 가지 소소한 장점을 주는데,
그건 해당 메서드가 `null`을 포함한 래퍼 클래스를 반환할 가능성이 있다는 점을 명시적으로 알려줄 수 있다는 것이다.
`Optional` 래퍼 클래스는 내부의 값이 `null`인지 검증하고 후속조치를 취하는 다양한 방법을 제공하므로, 호출자가 편하게 `null`을 처리할 수 있도록 도와준다.

### 2️⃣-3️⃣ 객체에 접근하기 전에는 null 체크부터
다른 개발자가 작성한 소스를 사용하거나 외부 라이브러리를 사용한 경우에는 1, 2번을 모두 준수해서 코드를 작성했을 것이라 확신할 수 없다.
따라서 이 경우에는, 메인 로직에서 `null` 검증 작업을 거처여 한다.

```java
public void updateUser(User user) {
    String phoneNumber = Optional.ofNullable(user)
            .map(user::getPhone)
            .map(Phone::getNumber)
            .orElseGet("번호 없음");
}
```
만약 `Optional`을 사용할 수 없다면, 일일이 검증을 하거나 예외를 던지는 수 밖에는 없다.

---

## 3️⃣ 언제 Optional을 사용해야 하는가?
Optional은 Null이 될 수 있는 객체를 감싸는 Wrapper 클래스이기 때문에 비용이 발생한다.
그래서 Optional은 필요한 경우에만 사용하는 것이 합리적이다.

`Optional`은 **null을 반환하면 오류가 발생할 가능성이 매우 높은 경우에 '결과 없음'을 명확하게 드러내기 위해 메서드의 반환 타입으로 사용되도록 매우 제한적인 경우로 설계되었다.**
이러한 설계 목적에 부합하게 실제로 Optional을 잘못 사용하면 많은 부작용(Side Effect)이 발생하게 된다.

### 3️⃣-1️⃣ Optional이 위험한 이유, Optional을 올바르게 사용해야 하는 이유
`Optional`을 사용하면 코드가 `Null-Safe`해지고, 가독성이 좋아지면 애플리케이션이 안정적이 된다는 등과 같은 얘기를 많이 접할 수 있다.
하지만 이는 `Optional`을 목적에 맞게 올바르게 사용했을 때의 이야기이고, `Optional`을 남발하는 코드는 오히려 다음과 같은 부작용(Side-Effect)을 유발할 수 있다.

> ⚠️ **NullPointerException 대신 NoSuchElementException가 발생**
> - 만약 Optional로 받은 변수를 값이 있는지 판단하지 않고 접근하려고 한다면 `NoSuchElementException`가 발생하게 된다.
> - `Null-Safe`하기 위해 `Optional`을 사용하였는데, 값의 존재 여부를 판단하지 않고 접근한다면 또 다른 예외가 발생한다.

> ⚠️ **이전에는 없었던 새로운 문제들이 발생**
> - `Optional`을 사용하면 문제가 되는 대표적인 경우가 직렬화(Serialize)를 할 때이다.
> 기본적으로 `Optional`은 직렬화를 지원하지 않아서 캐시나 메세지큐 등과 연동할 때 문제가 발생할 수 있다.
> - `Optional`을 사용함으로써 오히려 새로운 문제가 발생할 수 있다는 것이다.

> ⚠️ **코드의 가독성을 떨어뜨림**
> - 예를 들어 다음과 같이 Optional을 받아서 값을 꺼낸다고 하자. 우리는 앞에서 살펴본대로 optionAllUser의 값이 비어있으면 NoSuchElementException가 발생할 수 있으므로 다음과 같이 값의 유무를 검사하고 꺼내도록 코드를 작성하였다.
> ```java
> public void temp(Optional<User> optionalUser) {
>   User user = optionalUser.orElseThrow(IllegalStateException::new);
> 
> // 이후의 후처리 작업 진행...
> }  
> ```
> - 그런데 위와 같은 코드는 또 다시 NPE를 유발할 수 있다. 왜냐하면 optionalUser 객체 자체가 null일 수 있기 때문이다.
> - 그러면 우리는 이러한 문제를 해결하기 위해 다음과 같이 코드를 수정해야 한다.
> ```java
> public void temp(Optional<User> optionalUser) {
>   if (optionalUser != null && optionalUser.isPresent()) {
>       // 이후의 후처리 작업 진행... 
>   }
> 
>   throw new IllegalStateException();
> }  
> ```
> - 이러한 코드는 오히려 값의 유무를 2번 검사하게 하여 단순히 null을 사용했을 때보다 코드가 복잡해졌다.
> - 그 외에도 Optional의 제네릭 타입 때문에도 불필요하게 코드 글자수까지 늘어났다.
> 이렇듯 Optional을 올바르게 사용하지 못하고 남용하면 오히려 가독성이 떨어질 수 있다.

> ⚠️ **시간적, 공간적 비용(또는 오버헤드)이 증가**
> - 공간적 비용: Optional은 객체를 감싸는 컨테이너이므로 Optional 객체 자체를 저장하기 위한 메모리가 추가로 필요하다.
> - 시간적 비용: Optional 안에 있는 객체를 얻기 위해서는 Optional 객체를 통해 접근해야 하므로 접근 비용이 증가한다.
> 
> 어떤 글에서는 Optional을 사용하면 단순 값을 사용했을 때보다 메모리 사용량이 4배 정도 증가한다고 한다.
> 그 외에도 Optional은 Serializable 하지 않는 등의 문제가 있으므로 이를 해결하기 위해 추가적인 개발 시간이 소요돌 수 있다.
> 
> 하지만 위에서 설명한 예시들의 대부분은 Optional을 올바르게 사용하지 않았기 때문에 발생하는 것이다.
> Optional은 만들어진 의도가 상당히 명확한 만큼 목적에 맞게 올바르게 사용되어야 한다.

---

### 3️⃣-2️⃣ 올바른 Optional 사용법 가이드
- Optional 변수에 Null을 할당하지 말아라
- 값이 없을 때 Optional.orElseXxx()로 기본 값을 반환하라
- 단순히 값을 얻으려는 목적으로만 Optional을 사용하지 마라
- 생성자, 수정자, 메서드 파라미터 등으로 Optional을 넘기지 마라
- Collection의 경우 Optional이 아닌 빈 Collection을 사용하라
- 반환 타입으로 사용하라

#### Optional 변수에 Null을 할당하지 말아라
Optional은 컨테이너/박싱 클래스일 뿐이며, Optional 변수에 null을 할당하는 것은 Optional 변수 자체가 null인지 또 검사해야 하는 문제를 야기한다.
그러므로 값이 없는 경우라면 Optional.empty()로 초기화하도록 하자.

#### 값이 없을 때 Optional.orElseXxx()로 기본 값을 반환하라
Optional의 장점 중 하나는 함수형 인터페이스를 통해 가독성 좋고 유연한 코드를 작성할 수 있다는 것이다.
가급적이면 isPresent()로 검사하고 get()으로 값을 꺼내기 보다는 orElseGet 등을 활용해 처리하도록 하자.

```java
public String findUserName(long id) {
    Optional<String> optionalName = "...";
    return optionalName.orElseGet(this::findDefaultName);
}
```

orElseGet은 값이 준비되어 있지 않은 경우, orElse는 값이 준비되어 있는 경우에 사용하면 된다.
만약 null을 반환해야 하는 경우라면 orElse(null)을 활용하도록 하자.
만약 값이 없어서 throw 해야 하는 경우라면 orElseThrow를 사용하면 되고 그 외에도 다양한 메서드들이 있으니 적당히 활용하면 된다.

추가적으로 Java 9부터는 ifPresentOrElse도 지원하고 있고, Java 10부터는 orElseThrow()의 기본으로 NoSuchElementException()를 던질 수 있다.
만약 Java 8이나 9를 사용 중이라면 명시적으로 넘겨주면 되낟.

#### 단순히 값을 얻으려는 목적으로만 Optional을 사용하지 마라
단순히 값을 얻으려고 Optional을 사용하는 것은 Optional을 남용하는 대표적인 경우이다.
이러한 경우에는 굳이 Optional을 사용해 비용을 낭비하는 것보다는 직접 값을 다루는 것이 좋다.

```java
public String findUserName(long id) {
    String name = "...";
    
    return name == null ? "Default" : name;
}
```

#### 생성자, 수정자, 메서드 파라미터 등으로 Optional을 넘기지 마라
Optional을 파라미터로 넘기는 것은 상당히 의미없는 행동이다.
왜냐하면 넘겨온 파라미터를 위해 자체 null 체크도 추가로 해주어야 하고, 코드도 복잡해지는 등 상당히 번거로워지기 때문이다.

Optional은 반환 타입으로 대체 동작을 사용하기 위해 고안된 것임을 명심해야 하며, 앞서 설명한 대로 Serializable을 구현하지 않으므로 필드 값으로 사용하지 않아야 한다.

```java
// AVOID
public Optional<String> getName() {
    return Optional.ofNullable(name);
}
```
Optional을 접근자에 적용하는 경우도 마찬가지이다. 위의 예시에서 name을 얻기 위해 Optional.ofNullable()로 반환하고 있는데,
Brian Goetz는 Getter에 Optional을 얹어 반환하는 것을 두고 남용하는 것이라고 얘기하였다.
- Optional은 연산 결과(Result)를 감싸는 것
- Optional은 객체의 속성(Property)를 감싸는 게 아님
- 필드나 Getter에 Optional을 쓰는 것은 오해에서 비롯된 남용

```java
// PREFER
public Optional<User> findUserById(Long id) {
    // 찾았으면 Optional.of()
    // 없으면 Optional.empty()
}
```

#### Collection의 경우 Optional이 아닌 빈 Collection을 사용하라
Collection의 경우 굳이 Optional로 감쌀 필요가 없다.
오히려 빈 Collection을 사용하는 것이 깔끔하고, 처리가 가볍다.

```java
// AVOID
public Optional<List<User>> getUserList() {
    List<User> userList = null; // null이 올 수 있음
    
    return Optional.ofNullable(userList);
}

// PREFER
public Optional<List<User>> getUserList() {
    List<User> userList = null; // null이 올 수 있음
    
    return userList == null ? Collections.emptyList() : userList;
}
```

Optional은 비싸기 때문에 최대한 사용을 지양해야 한다. map을 사용한다면 getOrDefault() 메서드가 있으니 Optional을 사용하는 것보다 이걸 활용하는 것이 훨씬 좋다.

#### 반환 타입으로만 사용하라
Optional은 **반환 타입으로써 에러가 발생할 수 있는 경우에 결과 없음을 명확히 드러내기 위해** 만들어졌으며,
**Stream API와 결합되어 유연한 체이닝 api를 만들기 위해 탄생한 것이다.**

예를 들어, Stream API의 findFirst()나 findAny()로 값을 찾는 경우에 어떤 것을 반환하는게 합리적일지 Java 언어를 설계하는 사람이 되어 고민해보자.
언어를 만드는 사람의 입장에서 Null을 반환하는 것보다 값의 유무를 나타내는 객체를 반환하는 것이 합리적일 것이다.
Java 언어 설계자들은 이러한 고민 끝에 Optional을 만든 것이다.

그러므로 Optional이 설계된 목적에 맞게 반환 타입으로만 사용돠어야 한다.

---

## ☑️ 고민한 내용
VO 객체 테스트를 진행하면서 **null 검증의 중요성**을 다시 한 번 진지하게 생각하게 되었다.

내가 지금까지 구현한 코드에서는 **입력 단계에서만 null을 막아두면 충분**하다고 판단했고,
VO 내부에서는 별도로 null 검증을 하지 않았다.

어찌 보면 “내가 null을 넣지 않으면 된다”는 일종의 자신감이었던 것 같다.

하지만 테스트를 하면서 보니,
현재처럼 작은 규모의 프로젝트에서는 null로 인한 문제를 금방 파악할 수 있지만,
이 코드가 **여러 사람이 함께 사용하고, 더 많은 도메인과 로직이 얽히는 현실적인 환경**에서 동작한다고 가정하면 이야기가 완전히 달라진다.

“입력 단계에서만 검증하겠다”는 가정은 사실상 **모든 계층과 모든 개발자가 null을 절대 전달하지 않을 것이라고 믿는 것과 같다**.

이건 너무 위험한 전제이며, 실제로는 유지보수 과정에서 언제든 null이 유입될 가능성이 생긴다.

그래서 다시 고민하게 된 것이 “**VO 내부에서도 null 검증을 명확히 하는 것이 옳지 않을까**?” 라는 점이다.

VO는 불변 객체이며, 생성 시점에 값의 유효성을 보장해야 한다.

따라서 VO가 생성되는 순간 null을 철저히 차단하는 것이 도메인 모델의 신뢰성을 지키는 가장 단순하면서도 강력한 방법이라는 결론에 도달했다.

이후에는 Optional 사용 가이드와 NPE를 줄이는 여러 방법을 찾아보며 “애초에 null이 코드 흐름에 들어오지 않게 만드는 설계”가 가독성·안정성·확장성 모든 면에서 훨씬 낫다는 점을 깨닫게 되었다.

**즉, 입력 단계뿐 아니라 VO 내부에서도 null을 필수적으로 검증해야 하고,
가능한 null 자체가 애플리케이션 흐름에 등장하지 않도록 하는 방향으로 코드를 설계해야 한다**는 확신을 얻게 되었다.

---

## 🔖 References
- [자바 null 박살내기](https://blog.retrotv.dev/java-null-break/)
- [언제 Optional을 사용해야 하는가? 올바른 Optional 사용법 가이드](https://mangkyu.tistory.com/203?ref=blog.retrotv.dev)
