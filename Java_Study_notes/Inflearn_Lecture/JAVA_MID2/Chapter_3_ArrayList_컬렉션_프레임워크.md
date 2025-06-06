# 📚 (JAVA 인프런 - 중급 2편) Chapter 3: 컬렉션 프레임워크 - ArrayList
> 📌 공부 날짜: 2025/03/24

---

## ✅ 배열과 Index
- 배열과 같이 여러 데이터(자료)를 구조화해서 다루는 것을 `자료구조`라 한다.
- 자바는 배열 뿐만 아니라, `컬렉션 프레임워크`라는 이름으로 다양한 자료구조를 제공한다.

> 📚 배열의 경우 인덱스를 사용하면 한 번의 계산으로 매우 효율적으로 자료의 위치를 찾을 수 있다.

### 🔍 배열의 검색
- 배열에 들어있는 데이터를 찾는 것을 검색이라 한다.
- 배열에 들어있는 데이터를 하나하나 비교해야 한다. 인덱스를 사용해서 한 번에 찾을 수가 없다.
- **배열의 순차 검색은 배열에 들어있는 데이터의 크기 만큼 연산이 필요하다. 크기가 `n`이면 연산도 `n`만큼 필요함.**

---

## ✅ 빅오(O) 표기법
- 알고리즘의 성능을 분석할 때 사용하는 수학적 표현 방식이다.
- 여기서 중요한 것은 알고리즘의 정확한 실행 시간을 계산하는 것이 아니라, 데이터 양의 증가에 따른 성능의 변화 추세를 이해하는 것이다.

### 🔍 성능이 좋은 순
- `O(1)` → `O(log n)` → `O(n)` → `O(nlog n)` → `O(n^2)`

1. `O(1) - 상수 시간`: 입력 데이터의 크기에 관계없이 알고리즘의 실행 시간이 일정하다.
   - ex) 배열에서 인덱스를 사용하는 경우
   
2. `O(log n) - 로그 시간`: 알고리즘 실행 시간이 입력 데이터 크기의 로그에 비례하여 증가한다.
    - ex) 이진 탐색
   
3. `O(n) - 선형 시간`: 알고리즘 실행 시간이 입력 데이터 크기에 비례하여 증가한다.
    - ex) 배열의 검색, 배열의 모든 요소를 순회하는 경우
   
4. `O(nlog n) - 선형 로그 시간`: 많은 효율적인 정렬 알고리즘.

5. `O(n^2) - 제곱 시간`: 알고리즘 실행 시간이 입력 데이터 크기의 제곱에 비례하여 증가한다.
    - ex) 이중 루프 문

> 📚 `빅오 표기법`은 매우 큰 데이터를 입력한다고 가정한다.
> * 따라서 데이터가 매우 많이 들어오면 추세를 보는데 상수는 크게 의미가 없어진다.
> * 빅오 표기법에서 상수를 제거하며, `O(n/2)`, `O(n+2)`는 모두 `O(n)`으로 표시한다.
> * 별도의 표시가 없다면 최악의 상황을 가정해서 표기한다. 물론 최적, 평균, 최악의 경우로 나누어 표기하는 경우도 있다.

---

## ✅ 배열 데이터 추가
1. 배열의 첫 번째 위치에 추가 → `O(n)`만큼의 연산
2. 배열의 중간 위치에 추가 → `O(n/2)`만큼의 연산 즉 `O(n)`
3. 배열의 마지막 위치에 추가 → `O(1)` 한 번에 추가 가능.

- `1번`은 기존 데이터들을 모두 오른쪽으로 한 칸씩 밀어서 추가할 위치를 확보한다.
   - 이때 마지막 부분부터 오른쪽으로 밀어야 데이터를 유지할 수 있다.
---

## ✅ 배열의 한계
- 배열은 가장 기본적인 자료구조로 인덱스를 사용할 때 최고의 효율이 나온다.
- 하지만 이런 배열에는 큰 단점이 있다. 바로 배열의 크기를 생성하는 시점에 미리 정해야 한다는 점이다.

### 🔍 배열의 2가지 불편함
1. 배열의 길이를 동적으로 변경할 수 없다.
2. 데이터를 추가하기 어렵다.

> 📚 배열의 이러한 불편함을 해소하고 동적으로 데이터를 추가할 수 있는 자료구조를 `List`라고 한다.

---

## ✅ `List` 자료구조
**- 순서가 있고 중복을 허용하는 자료구조. 크기가 동적으로 변할 수 있다.**

> 📚 `Arrays.copyOf(배열, 새로운 길이)`: 새로운 길이로 배열을 생성하고, 배열의 값을 새로운 배열에 복사.

---

## ✅ 배열 리스트 `ArrayList`
- `List` 자료구조를 사용하는데, 내부의 데이터는 배열에 보관하는 것이다.

### 🔍 `ArrayList`의 빅오
1. 데이터 추가
   - 마지막에 추가: `O(1)`
   - 앞, 중간에 추가: `O(n)`
   
2. 데이터 삭제
   - 마지막 삭제: `O(1)`
   - 앞, 중간에 추가: `O(n)`
   
3. 인덱스 조회: `O(1)`

4. 데이터 검색: `O(n)`

> 📚 `ArrayList`는 보통 데이터를 중간에 추가하고 삭제하는 변경보다는,
> 데이터를 순서대로 입력하고 (데이터를 마지막에 추가하고), 순서대로 출력하는 경우에 가장 적합하다.

> 📚 `제네릭`은 `ArrayList`에 도입할 수 있는데 `Object`에 의한 타입 안정성 문제를 해결할 수 있다.
>  - 제네릭은 자료를 보관하는 자료구조에 가장 어울린다.

> 📚 `ArrayList` 내부에 제네릭을 사용했음에도 `Object` 배열을 그대로 사용한 이유는 타입 정보가 필요한 생성자에 제네릭을 사용할 수 없기 때문이다.
> 제네릭은 런타임에 이레이저에 의해 타입 정보가 사라진다. 자바가 제공하는 제네릭의 한계이다.

### 🔍 `ArrayList`의 단점
- 정확한 크기를 미리 알지 못하면 메모리가 낭비된다.
- 배열을 중간에 삭제하거나 추가할 때 비효율적이다. `O(n)` 성능으로 효율이 좋지 않다.

> 📚 이런 단점을 해결해주는 자료구조는 `링크드 리스트(LinkedList)`이다.
