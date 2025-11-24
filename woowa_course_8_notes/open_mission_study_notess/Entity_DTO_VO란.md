# 📖 지식 정리 (우테코를 하며 학습한 내용)
## 1️⃣ Entity란?
```text
Entity 클래스는 실제 DB의 Relation과 1:1로 매핑되는 클래스로,
DB Relation 내에 존재하는 attribute만 속성(필드)로 가져야 하는 것이 특징이다.

Domain Logic만을 가지고, Presentation Logic(View)를 가져서는 안 된다.
가급적 외부에서는 Entity 클래스의 필드에 직접적으로 접근하지 않도록 하고,
getter/setter보다 용도에 맞는 구체적인 메서드를 정의하여 값을 사용하도록 하는 것이 권장된다.

당연히 Request나 Response시 Entity 클래스를 직접 사용하는 것은 권장하지 않는다.
```

---

## 2️⃣ DTO란?
`DTO(Data Transfer Object)`란, 데이터 전송 객체의 준말이다.

계층 간 데이터 교환을 위해 사용하고,
- ex) `View <- (DTO) -> Controller <- (DTO) -> Service`

별도의 로직 없이 `getter/setter`만 존재한다.

DTO는 주로 비동기 처리를 위해 사용한다.

> 🤔 왜 Entity가 있는데 DTO를 따로 만들어 쓸까?
> - 그 이유는 DB(Persistence Layer)와 View(Presentation Layer)의 명확한 분리를 위해서이다.
> - Entity는 테이블과 매핑되므로, 변경되면 다른 클래스에 영향을 끼치기가 쉽다.
> - 또한, DTO는 주로 View와 통신하며 자주 변경되므로 분리가 필요하다.
> - DTO는 도메인 모델 객체인 Entity를 복사한 후, Presentation Logic만을 추가할 뿐이다.

---

## 3️⃣ VO란?
`VO(Value Object)`는 값 객체의 의미를 가진다. 특정 비즈니스 값을 담아낸다.

언뜨 보면 DTO와 비슷하지만, Read Only 속성을 가진다.

모든 속성 값이 같다면 같은 객체임을 의미한다.

생성자를 제외하고 set 성격의 메서드를 사용할 수 없다.

**VO 내부에 선언된 필드 내 모든 값들이 같은 경우, 동일한 객체라고 판단한다.**
- 이를 위해 equals()와 hashCode() 메서드를 Override해서 각 객체의 **동일성**을 판별할 수 있다.

📌 정리
1. **불변성**
- Value Object는 불변하다. 즉 수정자(setter)가 없다.
- 불변하기 때문에 존재하는 장점은 참조를 통해 공유해도 이로 인한 Side Effect가 없다.

2. **값 동등성**
- 두 Value Object가 동일한 값을 가진다면 둘은 동등하다. 이를 위해 equals와 hashcode 메서드를 재정의한다.

3. **자가 유효성 검증**
- Value Object는 유효한 값으로만 생성된다. 즉 아래와 같은 Age VO가 있을 때 생성된 모든 Age 값 객체는 0이상을 보장할 수 있다.

```java
public class Age {
    private int value;
    
    public Age (int value) {
        validate(value);
        this.value = value;
    }
    
    public void validate(int value) {
        if (value < 0) {
            // 예외 발생
        }
    }
}
```

### 테스트 코드 작성 시 VO 활용 팁
1. **Fixture 활용**: 테스트 전용 공통 VO 인스턴스를 미리 정의해두면, 반복적인 데이터 셋업 코드를 줄이고 핵심 테스트 로직에 집중할 수 있다.

2. **빌더 패턴(Builder Pattern) 고려**: 생성자의 인자가 많아지면 테스트 코드 작성이 복잡해질 수 있다. 이 경우, 빌더 패턴을 사용하여 필요한 값만 설정하여 VO 인스턴스를 유연하게 생성할 수 있다.

3. **테스트 데이터 관리 효율화**: 복잡한 데이터 구조의 경우, JSON 파일이나 SQL 스클립트를 활용하여 테스트 데이터를 효율적으로 관리할 수 있다.

4. **명확한 테스트 의도**: VO를 사용하면 비즈니스 행위나 의도를 테스트 이름이나 코드에 더 명확하게 담을 수 있다. 예를 들어, 단순히 문자열을 검증하는 대신 `Email` VO의 `isValidFormat() 메서드를 테스트하는 식으로 작성한다.

---

> 🤔 그래서 DTO랑 VO는 무슨 차이일까?
> - DTO는 값을 전달하는 것이 주 목적이고, VO는 값을 표현하는 것이 주 목적이다.
>   - 외부 시스템과 데이터 통신을 할 경우 DTO로, DB에서 가져오는 Data는 VO로 정의 후 사용
> - 그리고 DTO는 속성 값이 같아도 서로 다른 객체라고 판단하지만, VO는 같은 객체로 취급한다.
> - 마지막으로, DTO는 값의 전달이 그 목적이기 때문에 getter/setter를 사용할 수 있지만,
>   - VO는 불변성을 보장하기 위해 setter를 사용하지 못한다.

---

## ☑️ 고민한 내용
VO를 설계할 때 가장 고민됐던 부분은 **`equals()`와` hashCode()`를 반드시 오버라이딩해야 하는가**?라는 점이었다.

VO는 “값이 같으면 같은 객체로 취급한다”는 개념을 갖고 있으므로, `Lotto` 번호처럼 값 자체가 중요한 도메인에서는 두 객체가 같은 값을 가지면 동일한 로또 번호로 보아야 한다는 직관이 들었다.

만약 `equals()`와 `hashCode()`를 오버라이딩하지 않으면, 두 VO의 내부 값이 완전히 같더라도 서로 다른 객체로 판단하게 된다.

이는 VO의 목적(값을 기준으로 동등성을 판단하는 객체)과 맞지 않기 때문에, 결국 VO를 VO답게 사용하려면 `equals()`와 `hashCode()`를 재정의하는 것이 맞다는 결론에 도달했다.

즉, `LottoNumber`처럼 “**값 자체가 도메인의 핵심 개념이 되는 경우**”에는 오버라이딩을 통해 값 기반 비교를 명확하게 보장하는 것이 훨씬 안전하고 일관성 있는 설계라고 판단했다.

---

## 🔖 References
- [[Spring] Entity, DTO, VO 무슨 차이야?](https://youwjune.tistory.com/39)
- [VO (Value Object) 정리](https://medium.com/sjk5766/vo-value-object-%EC%A0%95%EB%A6%AC-63c207aa39f6)
