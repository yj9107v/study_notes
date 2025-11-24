# 📖 지식 정리 (우테코를 하며 학습한 내용)
## 1️⃣ DI란 무엇인가
`DI(Dependency Injection)`, **의존관계 주입**이라는 말로 사용한다.

먼저 Dependency, 의존관계에 대해 알아보자.

---

### 1️⃣-1️⃣ Dependency 의존관계란 무엇인가?
"A가 B를 의존한다."는 표현은 어떤 의미일까?
> 의존대상 B가 변하면, 그것이 A에 영향을 미친다.
> - 이일민, 토비의 스프링3.1, 에이콘(2021), p113

즉, B의 기능이 추가 또는 변경되거나 형식이 바뀌면 그 영향이 A에 미친다.

```java
class BurgerChef {
    private BurgerRecipe burgerRecipe;
    
    public BurgerChef() {
        burgerRecipe = new HamBurgerRecipe(); 
//        burgerRecipe = new CheeseBurgerRecipe(); 
//        burgerRecipe = new ChickenBurgerRecipe(); 
    }
}

interface BurgerRecipe {
    newBurger();
    // 이외의 다양한 메소드
}

class HamBurgerRecipe implements BurgerRecipe {
    public Burger newBurger() {
        return new HamBerger();
    }
    // ...
}
```
**의존관계를 인터페이스로 추상화하게 되면**, 더 다양한 의존관계를 맺을 수가 있고, 실제 구현 클래스와의 관계가 느슨해지고, 결합도가 낮아진다.

---

### 1️⃣-2️⃣ 그렇다면 Dependency Injection은?
의존관계를 외부에서 결정하고 주입하는 것이 **DI(의존관계 주입)**이다.

> 토비의 스프링에서는 다음의 세 가지 조건을 충족하는 작업을 의존관계 주입이라 말한다.
> - 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스만 의존하고 있어야 한다.
> - 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.
> - 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.
>   - **이일민, 토비의 스프링3.1, 에이콘(2012), p114**

---

## 1️⃣-3️⃣ DI 구현 방법
DI는 의존관계를 외부에서 결정하는 것이기 때문에, 클래스 변수를 결정하는 방법들이 곧 DI를 구현하는 방법이다.
런타임 시점의 의존관계를 외부에서 주입하여 DI 구현이 완성된다.

필드, setter, 생성자 세 가지가 있지만 스프링 4.x 기준으로 생성자가 기준이 되었다.

```java
// 생성자 이용
class BurgerChef() {
    private BurgerRecipe burgerRecipe;
    
    public BurgerChef(BurgerRecipe burgerRecipe) {
        this.burgerRecipe = burgerRecipe;
    } 
}

class BurgerRestaurantOwner {
    private BurgerChef burgerChef = new BurgerChef(new HamBurgerRecipe());
    
    public void changeMenu() {
        burgerChef = new BurgerChef(new CheeseBurgerRecipe());
    }
}
```

---

### 1️⃣-4️⃣ DI 장점
그렇다면, DI 의존관계를 분리하여 주입을 받는 방법의 코드 구현은 어떠한 장점이 있을까?

#### 1. **의존성이 줄어든다.**
앞서 설명했듯이, 의존한다는 것은 그 의존대상의 변화에 취약하다는 것이다.
(대상이 변화하였을 때, 이에 맞게 수정해야 함)

DI로 구현하게 되었을 때, 주입받는 대상이 변하더라도 그 구현 자체를 수정할 일이 없거나 줄어들게 됨.

#### 2. **재사용성이 높은 코드가 된다.**
기존에 BurgerChecf 내부에서만 사용되었던 BurgerRecipe을 별도로 구분하여 구현하면, 다른 클래스에서 재사용할 수가 있다.

#### 3. **테스트하기 좋은 코드가 된다.**
BurgerRecipe의 테스트를 BurgerChef 테스트와 분리하여 진행할 수 있다.

#### 4. **가독성이 높아진다.**
BurgerRecipe의 기능들을 별도로 분리하게 되어 자연스레 가독성이 높아진다.

---

## ☑️ 느낀 점
- DI란 개념을 명확히 알게 되었고, 어느 상황에서 사용해야 하는지 확실히 이해하게 되었다.

---

## 🔖 References
- [의존관계 주입(Dependency Injection) 쉽게 이해하기](https://tecoble.techcourse.co.kr/post/2021-04-27-dependency-injection/)
