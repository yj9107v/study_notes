# 📖 지식 정리 (우테코를 하며 학습한 내용)
## 1️⃣ record란 무엇일까?
`record`는 Java 14에서 처음 소개된 새로운 클래스 타입으로, 변경 불가(immutable) 데이터 객체를 쉽게 만들 수 있게 한다.

또한 `record`는 기존의 클래스와 비슷하지만, 더 간결하고 효율적으로 데이터 객체를 생성할 수 있도록 설계되어 있다.

---

## 2️⃣ record 특징

> **간결성**
> - `record`는 코드를 간결하게 만들기 위한 목적으로 도입되었다.
> - 필드를 정의하면 해당 필드를 기반으로 자동으로 메서드가 생성되어 코드의 양을 줄일 수 있고, 불필요한 보일러플레이트 코드를 줄여 가독성을 높여준다.
>   - `보일러플레이트`: 최소한의 변경으로 여러 곳에서 재사용되며, 반복적으로 비슷한 형태를 띄는 코드를 의미.

> **메서드 자동 생성(Automatic Method Generation)**
> - `record`는 필드를 기반으로 `equals()`, `hashcode()`, `toString()` 메서드를 자동으로 생성한다.
> - 이로써 불필요한 반복적인 코드 작성을 피할 수 있다.

> **생성자 자동 생성(Automatic Constructor Generation)**
> - `record`는 필드를 기반으로 자동으로 생성자를 생성한다.
> - 그러나 필요에 따라 명시적인 생성자를 정의할 수도 있다.

> **불변성(Immutability)**
> - `record`는 필드가 한 번 설정되면 값을 변경할 수 없다.
> - 이는 불변성을 보장하며 데이터의 안정성을 높이는데 도움이 된다.

> **final 선언 생략**
> - `record`의 필드를 final로 선언할 필요가 없다. Compiler는 필드를 불변으로 취급하고 자동으로 final로 처리한다.

> **패턴 매칭(Pattern Matching) 통합**
> - `record`는 Java의 Pattern Matching과 함께 사용될 수 있어, 데이터 추출 및 패턴 일치 검사에 효과적으로 사용될 수 있다.

> **데이터 전달(Data Transfer)**
> - `record`의 주된 목적은 객체 간에 불변 데이터를 전달하는 것이다. 따라서 `DTO(Data Transfer Object)`를 표현하는데 적합하다.

---

## 3️⃣ record 제약사항
- `abstarct`일 수 없어 다른 클래스를 **상속받을 수 없다**.
- 한 번 값이 정해지면 `setter`를 통해 값을 변경할 수 없다.
- 레코드 내부에 `private final` 이외의 멤버 변수(인스턴스 필드)를 선언할 수 없다.
  - 그러나 `static 변수`는 생성이 가능하다. 이는 헤더에서 정의한 멤버만을 record에서 관리하기 위함이다.

---

## 🔖 References
- [Record란? - 네이버 블로그](https://m.blog.naver.com/seek316/223341255150)
- [Java Record 란? - 개발로 자기개발 - 티스토리](https://ivory-room.tistory.com/92)
