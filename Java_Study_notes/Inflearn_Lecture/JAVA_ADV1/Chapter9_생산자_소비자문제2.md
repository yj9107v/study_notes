# 📚 Chapter 9: 생산자 소비자 문제 2

> 📌 공부 날짜: 2025/04/25
> - `JAVA `- 고급 1편
> - `Reference`: 인프런 - 김영한의 실전 자바

---
- 생산자가 생산자를 깨우고, 소비자가 소비자를 깨우는 비효율 문제를 어떻게 해결할 수 있을까?

## ✅ Lock Condition
- synchronized 대신에 Lock lock = new ReentrantLick();을 사용한다.

### 📚 Condition
```java
Condition condition = lock.newCondition();
```
- Condition은 ReentrantLock을 사용하는 스레드가 대기하는 스레드 대기 공간이다.
- lock.newCondition을 호출하면 스레드 대기 공간이 만들어진다.

> 🔍참고
> - Object.wait()에서 사용한 스레드 대기 공간은 모든 객체 인스턴스가 내부에 기본으로 가지고 있다.
> - 반면에 Lock(ReentrantLock)을 사용하는 경우 이렇게 직접 스레드 대기 공간을 만들어야 한다.

1. condition.await()
    - Object.wait()와 유사한 기능이다. 지정한 condition에 현재 스레드를 대기(Waiting) 상태로 보관한다.
    - 이때 ReentrantLock에서 획득한 락을 반납하고(모니터 락 아님) 대기 상태로 condition에 보관된다.

2. condition.signal()
    - Object.notify()와 유사한 기능이다. 지정한 condition에서 대기 중인 스레드를 하나 깨운다.
    - 깨어난 스레드는 condition에서 빠져나온다.

---

### 📚 Condition 분리 (비효율 문제 해결)
```java
Condition producerCond = lock.newCondition();
Condition consumerCond = lock.newCondition();
```
- 소비자를 위한 스레드 대기 공간과 생산자를 위한 스레드 대기 공간을 만들어 정확하게 나누어 관리하고 깨울 수 있다.
- **여기서 핵심은 생산자는 소비자를 깨우고, 소비자는 생산자를 깨우는 것이다.** 

---

### 📚 Object.notify() vs Condition.signal()
1. Object.notify()
   - 대기 중인 스레드 중 임의의 하나를 선택해서 깨운다.
   - 스레드가 깨어나는 순서는 정의되어 있지 않으며, JVM 구현에 따라 다르다.
   - 보통은 먼저 들어온 스레드가 먼저 수행되지만 구현에 따라 다를 수 있다.
   - synchronized 블록 내에서 모니터 락을 가지고 있는 스레드가 호출해야 한다.

2. Condition.signal()
   - 대기 중인 스레드를 하나 깨우며, 일반적으로는 FIFO 순서로 깨운다.
   - 이 부분은 자바 버전과 구현에 따라 달라질 수 있지만, 보통 Condition의 구현은 Queue 구조를 사용하기 때문에 FIFO 순서로 깨운다.
   - ReentrantLock을 가지고 있는 스레드가 호출해야 한다.

---

### 📚 스레드의 대기
#### 1. synchronized 2가지 단계의 대기 상태가 존재한다.

1. 모니터 락 획득 대기
   - 자바 객체 내부의 락 대기 집합(모니터 락 대기 집합)에서 관리
   - BLOCKED 상태로 락 획득 대기
   - synchronized를 시작할 때 락이 없으면 대기

2. wait() 대기
   - wait()을 호출할 때 자바 객체 내부의 스레드 대기 집합에서 관리
   - WAITING 상태로 대기
   - 다른 스레드가 notify()를 호출했을 때 스레드 대기 집합을 빠져나감.

#### 📌정리
> 🔍 자바의 모든 객체 인스턴스는 멀티스레드와 임계 영역을 다루기 위해 내부에 3가지 기본 요소를 가진다.
>   - 모니터 락
>   - 락 대기 집합(모니터 락 대기 집합)
>   - 스레드 대기 집합
- 여기서 락 대기 집합이 1차 대기소이고, 스레드 대기 집합이 2차 대기소라 생각하면 된다.
- 2차 대기소에 들어간 스레드는 1차 대기소까지 빠져나와야 임계 영역을 수행할 수 있다.
- 이 3가지 요소는 서로 맞물려 돌아간다.
  - synchronized를 사용한 임계 영역에 들어가려면 모니터 락이 필요하다.
  - 모니터 락이 없으면 락 대기 집합에 들어가서 BLOCKED 상태로 락을 기다린다.
  - 모니터 락을 반납하면 락 대기 집합에 있는 스레드 중 하나가 락을 획득하고 BLOCKED -> RUNNABLE 상태가 된다.
  - wait()를 호출해서 스레드 대기 집합에 들어가기 위해서는 모니터 락이 필요하다.
  - 스레드 대기 집합에 들어가면 모니터 락을 반납한다.
  - 스레드가 notify()를 호출하면 스레드 대기 집합에 있는 스레드 중 하나가 스레드 대기 집합을 빠져나온다. 그리고 모니터 락 획득을 시도한다.
    - 모니터 락을 획득하면 임계 영역을 수행한다.
    - 모니터 락을 획득하지 못하면 락 대기 집합에 들어가서 BLOCKED 상태로 락을 기다린다.

---

#### 2. Lock(ReentrantLOck) 또한 2가지 단계의 대기 상태가 존재한다.
1. ReentrantLock 락 획득 대기
   - ReentrantLock의 대기 큐에서 관리
   - WAITING 상태로 락 획득 대기
   - lock.lock()을 호출했을 때 락이 없으면 대기
   - 다른 스레드가 lock.unLock()을 호출했을 때 대기가 풀리며 락 획득 시도, 락을 획득하면 대기 큐를 빠져나감.

2. await() 대기
   - Condition.await()를 호출했을 때, Condition 객체의 스레드 대기 공간에서 관리
   - WAITING 상태로 대기
   - 다른 스레드가 Condition.signal()을 호출했을 때, Condition 객체의 스레드 대기 공간에서 빠져나감.

---

## ✅ BlockingQueue
- 스레드 관점에서 보면 큐가 특정 조건이 만족될 때까지 스레드의 작업을 차단(Blocking)한다.
   1. 데이터 추가 차단: 큐가 가득 차면 데이터 추가 작업(put())을 시도하는 스레드는 공간이 생길 때까지 차단된다.
  2. 데이터 획득 차단: 큐가 비어있으면 획득 작업(take())을 시도하는 스레드는 큐에 데이터가 들어올 때까지 차단된다.

- **자바는 생산자 소비자 문제, 또는 한정된 버퍼라고 불리는 문제를 해결하기 위해 java.util.concurrent.BlockingQueue라는 인터페이스와 구현체를 제공한다.**

### 📚 BlockingQueue 인터페이스의 대표적인 구현체
1. ArrayBlockingQueue: 배열 기반으로 구현되어 있고, 버퍼의 크기가 고정되어 있다.
2. LinkedBlockingQueue: 링크 기반으로 구현되어 있고, 버퍼의 크기를 고정할 수도, 또는 무한하게 사용할 수도 있다.

- 1은 내부에서 ReentrantLock을 사용한다. 그리고 생산자 전용 대기실과 소비자 전용 대기실이 있다.

---

### 📚 BlockingQueue - 기능 설명
- 실무에서 멀티스레드를 사용할 때는 응답성이 중요하다.
- 예를 들어, 대기 상태에 있어도, 고객이 중지 요청을 하거나, 또는 너무 오래 대기한 경우 포기하고 빠져나갈 수 있는 방법이 필요하다.
  - 다양한 상황이 있기에 BlockingQueue는 각 상황에 맞는 다양한 메서드를 제공한다.

> 큐가 가득 찼을 때 생각할 수 있는 선택지는 4가지다.
> - 예외를 던진다. 예외를 받아서 처리한다.
> - 대기하지 않는다. 즉시 false를 반환한다.
> - 대기한다.
> - 특정 시간만큼만 대기한다.

1. Throws Exception - 대기 시 예외
   - add(e): 지정된 요소를 큐에 추가하며, 큐가 가득 차면 IllegalStateException 예외를 던진다.
   - remove(e): 큐에서 요소를 제거한다. 큐가 비어있으면 NoSuchElementException 예외를 던진다.
   - element(e): 큐의 머리 요소를 반환하지만, 요소를 큐에서 제거하지 않는다. 큐가 비어있으면 NoSuchElementException 예외를 던진다.

2. Special Value - 대기 시 즉시 반환
   - offer(e): 지정된 요소를 큐에 추가하려고 시도하며, 큐가 가득 차면 false를 반환한다.
   - poll(): 큐에서 요소를 제거하고 반환한다. 큐가 비어있으면 null을 반환한다.
   - peek(): 큐의 머리 요소를 반환하지만, 요소를 큐에서 제거하지 않는다. 큐가 비어있으면 null을 반환한다.

3. Blocks - 대기
   - put(e): 지정된 요소를 큐에 추가할 때까지 대기한다. 큐가 가득 차면 공간이 생길 때까지 대기한다.
   - take(): 큐에서 요소를 제거하고 반환한다. 큐가 비어있으면 요소가 준비될 때까지 대기한다.
   - Examine(관찰): 해당 사항 없음.

4. Times Out - 시간 대기
   - offer(e, time, unit): 지명된 요소를 큐에 추가하려고 시도하며, 지정된 시간 동안 큐가 비워지기를 기다리다가 시간이 초과되면 false를 반환한다.
   - poll(time, unit)
   - Examine(관찰): 해당 사항 없음.

> 🔍참고
> - BlockingQueue의 모든 대기, 시간 대기 메서드는 인터럽트를 제공한다.

- ❗핵심: 실무에서 사용할 때에는 이제 BlockingQueue를 사용하면 된다.

---
