# 📚 Chapter 10: 컬렉션 프레임워크 - 순회, 정렬 전체 정리

> 📌 공부 날짜: 2025/04/04
> - `JAVA `- 중급 2편
> - `Reference`: 인프런 - 김영한의 실전 자바

---

## ✅ 순회1 - Iterable, Iterator
- `순회`라는 단어는 여러 곳을 돌아다닌다는 뜻이다.
- 자료구조의 순회는 자료구조에 들어있는 데이터를 차례대로 접근해서 처리하는 것을 `순회`라 한다.
- 다양한 자료구조가 있으며, 각각의 자료구조마다 데이터를 접근하는 방법이 모두 다르다.
- 자료구조의 구현과 관계 없이 모든 자료구조를 동일한 방법으로 순회할 수 있는 일관성 있는 방법이 바로 `Iterable`, `Iterator` 인터페이스이다.

  ### 📚 `Iterable`의 주요 메서드
  - 단순히 Iterator 반복자를 반환한다.

  ### 📚 `Iterator`의 주요 메서드
  - **hashNext()**: 다음 요소가 있는지 확인한다. 다음 요소가 없으면 `false`를 반환.
  - **next()**: 다음 요소를 반환한다. 내부에 있는 위치를 다음으로 이동한다.

---

## ✅ 순회2 - 향상된 `for문`
- `Iterable`과 향상된 `for문`(Enhanced For Loop)
  - `for-each`문으로 불리는 향상된 `for문`은 자료구조를 순회하는 것이 목적이다.
```java
ex)
    for (int value: myArr) {
        System.out.println(value);
    }

ex)
    while (iteraotr.hashNext()) {
        System.out.println(iterator.next());
    }
```
- 둘 다 똑같은 코드이다. 모든 데이터를 순회한다면 향상된 `for문`을 사용하는 것이 좋다.

> 🔍 정리
> - 만드는 사람이 수고로우면 쓰는 사람이 편하고, 만드는 사람이 편하면 쓰는 사람이 수고롭다.
> - 특정 자료구조가 `Iterable`, `Iterator`를 구현한다면, 해당 자료구조를 사용하는 개발자는 단순히 hashNext(), next() 또는 `for-each`문을 사용해서 순회할 수 있다.
> - 자료구조가 아무리 복잡해도 해당 자료구조를 사용하는 개발자는 동일한 방법으로 매우 쉽게 자료구조를 순회할 수 있다.
> - 이것이 인터페이스가 주는 큰 장점이다.

---

## 순회3 - 자바가 제공하는 `Iterable`과 `Iterator`
- 자바가 제공하는 컬렉션 프레임워크의 모든 자료구조는 `Iterable`과 `Iterator`를 사용해서 편리하고 일관된 방법으로 순회할 수 있다.
  - 향상된 `for문`도 사용 가능.
```java
    private static void forEach(Iterable<Integer> iterable) {
        for (Integer i : iterable) {
            System.out.println(i);
        }
    }
```
- 리스트, 셋 등 `forEach(list.iterator())`, `forEach(set.iterator())` 처럼 다형성을 적극 확용할 수 있다.

> 🔍 참고
> - **`Iterator`(반복자) 디자인 패턴**은 객체 지향 프로그래밍에서 컬렉션의 요소들을 순회할 때 사용된 디자인 패턴이다.
> - 이 패턴은 내부 표현 방식을 노출시키지 않으면서도 그 안의 각 요소에 순차적으로 접근할 수 있게 해준다.
> - `Iterator` 패턴은 컬렉션의 구현과는 독립적으로 요소들을 탐색할 수 있는 방법을 제공하며, 이로 인해 코드의 복잡성을 줄이고 재사용성을 높일 수 있다.

---

## ✅ 정렬1 - Comparable, Comparator
  ### 📚 비교자 Comparator
  ```java
  public interface Comparator<T> {
      int compare (T1, T2)
  }
  ```
  - 두 인수를 비교해서 결과 값을 반환하면 된다.
    - 첫 번째 인수가 더 작으면 음수 (-1)
    - 두 값이 같으면 (0)
    - 첫 번째 인수가 더 크면 양수 (1)
    
  ```java
  static class AscComparator implements Comparator<Integer> {
      @Override
      public int compare(Integer o1, Integer o2) {
          return (o1 < o2) ? -1 : ((o1 == o2) ? 0 : 1);
      }
  }
  ```
  - Arrays.sort()를 할 때 비교자(Comparator)를 넘겨주면 알고리즘에서 어떤 값이 더 큰지 두 값을 비교할 때 비교자를 사용한다.
  ```java
      Arrays.sort(array, new AscComparator());
  ```
  - `AscComparator`가 오름차순인데 반환 값에 -1을 곱해주면 반대인 내림차순 정렬로 만들 수 있다.
    - 정렬을 반대로 하고 싶다면 reversed() 메서들르 사용해도 된다.

---

## ✅ 정렬2 - Comparable, Comparator
- 자바가 기본으로 제공하는 Integer, String 같은 객체를 제외하고 MyUser 와 같이 직접 만든 객체를 정렬하려면 어떻게 해야 하나?
  - 이때는 Comparable 인터페이스를 구현하면 된다. 객체에 비교 기능을 추가해 준다.
  - `Comparable`을 통해 구현한 순서를 자연 순서(Natural Ordering)라 한다.

  ### 📚 다른 방식으로 정렬
  - 만약 객체가 가지고 있는 `Comparable` 기본 정렬이 아니라 다른 정렬을 사용하고 싶다면 어떻게 할까?
  ```java
    public class IdComparator implements Comparator<MyUser> {
      
    @Override
    public int compare(MyUser o1, MyUser o2) {
      return o1.getId().compareTo(o2.getId());
    }
  }
  ```
  - `Arrays.sort`의 인수로 비교자(Comparator)를 만들어서 넘겨주면 된다.
  
  > 🚨 주의
  > - 만약 `Comparable`도 구현하지 않고, `Comparator`도 제공하지 않으면 런타임 오류 발생.
  
---

## ✅ 정렬3 - Comparable, Comparator
- 정렬은 배열 뿐만 아니라 순서가 있는 List 같은 자료구조에도 사용할 수 있다.

  ### 📚 Collections.sort(list)
  - 리스트는 순서가 있는 컬렉션이므로 정렬할 수 있다.
    - 하지만 이 방식 보다는 객체 스스로 정렬 메서드를 가지고 있는 `list.sort()` 사용을 더 권장한다.

  ### 📚 list.sort(null)
  - 별도의 비교자가 없으므로 `Comparable`로 비교해서 정렬한다.
  - 비교자 없을 때에는 `null`이라도 넣어주어야 한다.

  ### 📚 TresSet 구조와 정렬
  - 이진 탐색 트리 구조는 데이터를 보관할 때, 데이터를 정렬하면서 보관한다.
    - 따라서 정렬 기준 제공하는 것이 필수다.
  - 이진 탐색 트리는 데이터를 저장할 때 왼쪽 노드에 저장할지 오른쪽 노드에 저장할지 비교가 필요하다.
    - 따라서 `TreeSet`과 `TreeMap`은 Comparable 또는 Comparator 가 필수이다.
  ```java
    // TreeSet을 생성할 때 별도의 비교자를 제공하거나 하지 않는 방버이 있다.
    new TreeSet<>(); // or
    new TreeSet<>(new IdComparator());
  ```

---

## ✅ 컬렉션 유틸
- 컬렉션은 편리하게 다룰 수 있는 기능을 알아보자.
  - ex) max, min, shuffle, reverse, sort

  ### 📚 편리한 컬렉션 성능
  - List.of(1,2,3)을 사용하면 컬렉션을 편리하게 생성할 수 있다.
  - 단 이때는 가변이 아니라 불변 컬렉션이 생성된다.
  
  ---
  ### 📚 불변 컬렉션과 가변 컬렉션 전환
  - 불변 리스트를 `가변 리스트`로 전환하려면 new ArrayList<>(list);를 사용하면 된다.
    - `list`는 불변.
  - 가변 리스트를 `불변 리스트`로 전환하려면 Collections.unmodifiableList()를 사용하면 된다.
  
  ---
  ### 📚 빈 불변 리스트 생성 2가지
  - Collections.emptyList();
    - 자바 5 이상
  - List.of() 이렇게 두 가지가 있지만 List.of()가 더 간결하고 List.of(1,2,3)도 불변이기 때문에 사용법에 일관성이 있다.
    - 자바 9 이상. 이 기능을 더 권장.
  
  ---
  ### 📚 멀티스레드 동기화
  - Collections.synchronizedList()를 사용하면 일반 리스트를 멀티스레드 상황에서 동기화 문제가 발생하지 않는 안전한 리스트로 만들 수 있다.
  - 동기화 작업으로 인해 일반 리스트보다 성능이 더 느리다.

  ---

  ### 🔍 컬렉션 프레임워크 전체 정리 
  - 자바 컬렉션 프레임워크는 데이터 그룹을 저장하고 처리하기 위한 통합 아키텍처를 제공한다.
  - 이 프레임워크는 인터페이스, 구현, 알고리즘으로 구성되어 있으며, 다양한 타입의 컬렉션을 효율적으로 처리할 수 있게 해준다.
    - 여기서 컬렉션이란 객체의 그룹이나 집합을 의미한다.

---

## ✅ Collection 인터페이스의 필요성
- 다양한 컬렉션 타입들이 공통적으로 따라야 하는 기본 규약을 정의한다.
- List, Set, Queue 와 같은 더 구체적인 컬렉션 인터페이스들은 모두 컬렉션 인터페이스를 확장하여,
공통된 메서드들을 추가로 상속받고 추가적인 기능이나 특성을 제공한다.
  - 이러한 설계는 자바 컬렉션 프레임워크의 일관성과 재사용성을 높여준다.
- 일관성, 재사용성, 확장성, 다형성
- 대표적으로 컬렉션 인터페이스는 `iterator`를 제공한다.
  - 모든 컬렉션의 타입의 데이터를 순회할 수 있다.

---

## 🤔 선택 가이드
  ### 1. 순서가 중요하고 중복이 허용되는 경우
  - List 인터페이스를 사용하자.
    - `ArrayList`가 일반적이지만, 추가/삭제 작업이 앞쪽에서 빈번한 경우에는 `LinkedList`가 성능상 더 좋은 선택이다.(데이터가 엄청 클시)

  ---
  ### 2. 중복을 허용하지 않고 순서가 중요하지 않는 경우
  - `HashSet`을 사용하자.
    - 순서를 유지해야 하면 `LinkedHashSet`을,
    - 정렬된 순서가 필요하면 `TreeSet`을 사용하자.

  ---
  ### 3. 요소를 키-값 쌍으로 저장하려는 경우
  - Map 인터페이스를 사용하자.
    - 순서가 중요하지 않다면 `HashMap`을,
    - 순서를 유지해야 한다면 `LinkedHashMap`을,
    - 정렬된 순서가 필요하면 `TreeMap`을 사용하자.

  ---
  ### 4. 요소를 처리하기 전에 보관해야 하는 경우
  - Queue, Deque 인터페이스를 사용하자.
    - 스택, 큐 구조 모두 `ArrayDeque`을 사용하는 것이 제일 빠르다.
    - 만약 우선순위에 따라 요소를 처리해야 한다면 `PriorityQueue`를 사용하면 된다.

  ---
  ### ❗ 실무 선택 가이드
  - `List`의 경우 대부분 `ArrayList`를 사용한다. (백엔드 개발자 시 95% 이상)
  - `Set`의 경우 대부분 `HashSet`을 사용한다.
  - `Map`의 경우 대부분 `HashMap`을 사용한다.
  - `Queue`의 경우 대부분 `ArrayDeque`을 사용한다.

---

