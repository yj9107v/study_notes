# 📚 Chapter 7: 고급 동기화 - concurrent_lock

> 📌 공부 날짜: 2025/04/22
> - `JAVA `- 고급 1편
> - `Reference`: 인프런 - 김영한의 실전 자바

---

## ✅ LockSupport
- synchronized의 단점인 무한 대기와 공정성의 문제를 해결할 더 유연하고 세밀한 제어가 가능한 방법들이 필요해졌다.
- 이런 문제를 해결하기 위해 자바 1.5부터 java.util.concurrent라는 동시성 문제 해결을 위한 라이브러리 패키지가 추가된다.
- 이 라이브러리에는 수많은 클래스가 있지만 가장 기본이 되는 LockSupport는 synchronized의 가장 큰 단점인 무한 대기 문제를 해결할 수 있다.

### 📚 LockSupport 기능
- LockSupport는 스레드를 WAITING 상태로 변경한다.
- WAITING 상태는 누가 깨워주기 전까지는 계속 대기한다. 그리고 CPU 실행 스케줄링 들어가지 않는다.
- `park()`: 스레드를 WAITING 상태로 변경한다.
- `parkNanos(nanos)`: 스레드를 나노초 동안만 TIME_WAITING 상태로 변경한다.
  - 지정한 나노초가 지나면 TIME_WAITING 상태에서 빠져나오고 RUNNABLE 상태로 변경된다.
- `unPark(thread)`: WAITING 상태의 대상 스레드를 RUNNABLE 상태로 변경한다.

> 🔍 인터럽트 사용
> - WAITING 상태의 스레드에 인터럽트가 발생하면 WAITING 상태에서 RUNNABLE 상태로 변하면서 깨어난다.

---

### 📚 BLOCKED vs WAITING
1. 인터럽트
  - BLOCKED 상태는 인터럽트가 걸려도 대기 상태를 빠져나오지 못한다. 여전히 BLOCKED 상태.
  - WAITING, TIME_WAITING 상태는 인터럽트가 걸리면 대기 상태를 빠져나온다.

2. 용도
  - BLOCKED 상태는 자바의 synchronized에서 락을 획득하기 위해 대기할 때 사용된다.
  - WAITING, TIME_WAITING 상태는 스레드가 특정 조건이나 시간 동안 대기할 때 발생하는 상태.

> 🔍 대기 상태(WAITING)와 시간 대기 상태(TIME_WAITING)는 서로 짝이 있다.
> - Thread.join(), Thread.join(long millis)
> - Thread.park(), Thread.parkNanos(long millis)
> - Object.wait(), Object.wait(long timeout)

- **BLOCKED, WAITING, TIME_WAITING 상태 모두 스레드가 대기하며, 실행 스케줄링에 들어가지 않기 때문에, CPU 입장에서 보면 실행하지 않는 비슷한 상태이다.**
- BLOCKED 상태는 synchronized에서만 사용하는 특별한 대기 상태라고 이해하면 된다.
- WAITING, TIME_WAITING 상태는 범용적으로 활용할 수 있는 대기 상태라고 이해하면 된다.

> 🤔 LockSupport는 너무 저수준이다. synchronized처럼 더 고수준의 기능이 필요하다.
> - 자바는 Lock 인터페이스와 ReentrantLock이라는 구현체로 이런 기능들을 이미 다 구현해 두었다.
> - ReentrantLock은 LockSupport를 활용해서 synchronized의 단점을 극복하면서도 매우 편리하게 임계 영역을 다룰 수 있는 기능을 제공한다.

---

## ✅ ReentrantLock 이론

### 📚 Lock 인터페이스
- 동시성 프로그래밍에서 쓰이는 민감한 임계 영역을 위한 락을 구현하는 데 사용된다.
- 아래와 같은 메서드를 제공한다. 대표적인 구현체로 ReentrantLock이 있다.

1. void lock()
  - 락을 획득한다. 만약 다른 스레드가 이미 락을 획득했다면, 락이 풀릴 때까지 현재 스레드는 대기(WAITING)한다. 이 메서드는 인터럽트에 응하지 않는다.

> 🚨 주의
> - 여기서 사용하는 락은 객체 내부에 있는 모니터 락이 아니다.
> - Lock 인터페이스와 ReentrantLock이 제공하는 기능이다.
> - 모니터 락과 BLOCKED 기능은 synchronized에서만 사용된다.

2. void lockInterruptibly()
  - 락 획득을 시도하되, 다른 스레드가 인터럽트 할 수 있도록 한다.
  - 만약 다른 스레드가 이미 락을 획득했더라면, 현재 스레드는 락을 획득할 때까지 대기한다. 대기 중에 인터럽트가 발생하면 InterruptedException이 발생하면 락 획득을 포기한다.

3. boolean tryLock()
  - 락 획득을 시도하고, 즉시 성공 여부를 반환한다.
  - 만약 다른 스레드가 이미 락을 획득했다면 false를 반환하고, 그렇지 않으면 락을 획득하고 true를 반환한다.

4. boolean tryLock(long time, TimeUnit unit)
  - 주어진 시간 동안 락 획득을 시도한다. 주어진 시간 안에 락을 획득하면 true를 반환한다.
  - 주어진 시간이 지나도 락을 획득하지 못한 경우에는 false를 반환한다.
  - 이 메서드는 대기 중에 인터럽트가 발생하면 예외가 발생하며 락 획득을 포기한다.

5. void unLock()
  - 락을 해제한다. 락을 해제하면 락 획득을 대기 중인 스레드 중 하나가 락을 획득할 수 있게 된다.
  - 락을 획득한 스레드가 호출해야 하며, 그렇지 않으면 IllegalMonitorStateException이 발생할 수 있다.

6. Condition newCondition()
  - Condition 객체를 생성하여 반환한다.
  - Condition 객체는 락과 결합되어 사용되며, 스레드가 특정 조건을 기다리거나 신호를 받을 수 있도록 한다.
  - 이는 Object 클래스의 wait, notify, notifyAll 메서드와 유사한 역할을 한다.

- **이 메서드들을 사용하면 고수준의 동기화 기법을 구현할 수 있다.**
- **Lock 인터페이스는 synchronized 블록보다 더 많은 유연성을 제공하며, 특히 락을 특정 시간만큼만 시도하거나, 인터럽트 가능한 락을 사용할 때 유용하다.**

---

### 📚 공정성
- Lock 인터페이스 덕분에 synchronized의 단점인 무한 대기 문제가 해결되었다.
- 남은 문제는 공정성이 있는데 Lock 인터페이스의 대표적인 구현체로 ReentrantLock이 있는데, 이 클래스는 스레드가 공정하게 락을 얻을 수 있는 모드를 제공한다.
- **ReentrantLock 락은 비공정(non-fair) 모드와 공정성(fairness) 모드로 설정할 수 있다.**

---

### 📚 비공정 모드 (Non-fair Mode)
```java
private final Lock nonFairLock = new ReentrantLock();
```
- 비공정 모드는 ReentrantLock의 기본 모드이다. 이 모드에서는 락을 먼저 요청한 스레드가 락을 먼저 획득한다는 보장이 없다.
- 락을 풀었을 때, 대기 중인 스레드 중 아무나 락을 획득할 수 있다.
- 이는 락을 빨리 획득할 수 있지만, 특정 스레드가 장기간 락을 획득하지 못할 가능성도 있다.

> 🔍 특징
> - 성능 우선
> - 선점 가능: 새로운 스레드가 기존 스레드보다 먼저 락을 획득할 수도 있다.
> - 기아 현상 가능성: 특정 스레드가 계속해서 락을 획득하지 못할 수 있다.

---

### 📚 공정 모드 (Fair Mode)
- 생성자에서 true를 전달하면 된다.
  - ex) new ReentrantLock(true);
- 공정 모드는 락을 요청한 순서대로 스레드가 락을 획득할 수 있게 한다.
- 이는 먼저 대기한 스레드가 먼저 락을 획득하게 되어 스레드 간의 공정성을 보장한다.
- 그러나 이로 인해 성능이 저하될 수 있다.

> 🔍 특징
> - 공정성 보장: 대기 큐에서 먼저 대기한 스레드가 락을 먼저 획득한다.
> - 기아 현상 방지: 모든 스레드가 언젠가 락을 획득할 수 있게 보장된다.
> - 성능 저하: 락을 획득하는 속도가 느려질 수 있다.

---

## ❗정리
- Lock 인터페이스와 ReentrantLock 구현체를 사용하면 synchronized 단점인 무한 대기와 공정성 문제를 모두 해결할 수 있다.

---

## 🤔 ReentrantLock 활용
- synchronized(this) 대신에 lock.lock()을 사용해서 락을 건다.
  - lock() -> unLock()까지는 안전한 임계 영역이 된다.
- 임계 영역이 끝나면 반드시 락을 반납해야 한다.
  - 따라서 unLock()은 반드시 finally() 블록에 작성해야 한다.

> 🔍참고
> - volatile을 사용하지 않아도 Lock을 사용할 때 접근하는 변수의 메모리 가시성 문제는 해결된다.

> ❗정리
> - 자바 1.5에서 등장한 Lock 인터페이스와 ReentrantLock 덕분에 synchronized의 단점인 무한 대기와 공정성 문제를 극복하고, 또 더욱 유연하고 세밀한 스레드 제어가 가능하다.

---