# 📚 Chapter 12_1: 스레드 풀과 Executor 프레임워크 1

> 📌 공부 날짜: 2025/05/01
> - `JAVA `- 고급 1편
> - `Reference`: 인프런 - 김영한의 실전 자바

---

## ✅ 스레드를 직접 사용할 때의 문제점

### 📚 1. 스레드 생성 비용으로 인한 성능 문제

- 스레드를 사용하려면 먼저 스레드를 생성해야 한다. 다음과 같은 이유로 매우 무겁다.

1. 메모리 할당
    - 각 스레드는 자신만의 호출 스택(call stack)을 가지고 있어야 한다.
    - 이 호출 스택은 스레드가 실행되는 동안 사용하는 메모리 공간이다.
    - 따라서 스레드를 생성할 때는 이 호출 스택을 위한 메모리를 할당해야 한다.

2. 운영체제 자원 사용
    - 스레드를 생성하는 작업은 운영체제 커널 수준에서 이루어지며, 시스템 콜을 통해 처리된다.
    - 이는 CPU와 메모리 리소스를 소모하는 작업이다.

3. 운영체제 스케줄러 조정
    - 새로운 스레드가 생성되면 운영체제의 스케줄러는 이 스레드를 관리하고 실행 순서를 조정해야 한다.
    - 이는 운영체제의 스케줄링 알고리즘에 따라 추가적인 오버헤드가 발생할 수 있다.

- **스레드를 생성하는 작업은 상대적으로 무겁다. 단순히 자바 객체를 하나 생성하는 것과는 비교할 수 없을 정도로 큰 작업이다.**
    - 아주 가벼운 작업이라면, 작업의 실행 시간보다 스레드의 생성 시간이 더 오래 걸릴 수도 있다.

> 🔍 1번 문제 해결
> - 이런 문제를 해결하려면 생성한 스레드를 재사용하는 방법을 고려할 수 있다.
> - 스레드를 재사용하면 처음 생성할 때를 제외하고는 생성을 위한 시간이 들지 않는다.
> - 따라서 스레드가 아주 빠르게 작업을 수행할 수 있다.

### 📚 2. 스레드 관리 문제
- 서버의 CPU, 메모리 자원은 한정되어 있기 때문에, 스레드는 무한하게 만들 수 없다.
- 우리 시스템이 버틸 수 있는, 최대 스레드의 수까지만 스레드를 생성할 수 있게 괸리해야 한다.

---

### 📚 3. Runnable 인터페이스의 불편함
- 반환 값이 없다.
    - run() 메서드는 반환 값을 가지지 않는다. 따라서 실행 결과를 얻기 위해서는 별도의 메커니즘을 사용해야 한다.
- 예외 처리
    - run() 메서드는 체크 예외(Checked exception)를 던질 수 없다.
    - 체크 예외의 처리는 메서드 내부에서 처리해야 한다.
- **이런 문제를 해결하려면 반환값도 받을 수 있고, 예외도 좀 더 쉽게 처리할 수 있는 방법이 필요하다. 해당 스레드에서 발생한 예외도 받을 수 있다면 좋을 것이다.**

---

### ❗ 해결
- 1번, 2번 문제를 해결하려면 스레드를 생성하고 관리하는 풀(pool)이 필요하다.
- 이런 문제를 한방에 해결해 주는 것이 바로 자바가 제공해 주는 `Executor` 프레임워크다.
    - Executor 프레임워크는 스레드 풀, 스레드 관리, Runnable의 문제점은 물론이고, 생산자 소비자 문제까지 한방에 해결해 주는 자바 멀티스레드 최고의 도구이다.
    - 3번 문제를 해결해 주는 Executor 프레임워크는 Callable과 Future라는 인터페이스를 도입했다.
- **실무에서는 스레드를 직접 생성해서 사용하는 것보다는 Executor 프레임워크를 주로 사용한다.**

---

## ✅ Executor 프레임워크 소개
- 멀티스레딩 및 병렬 처리를 쉽게 사용할 수 있도록 돕는 기능의 모음이다.
- 이 프레임워크는 작업 실행의 관리 및 스레드 풀 관리를 효율적으로 처리해서 개발자가 직접 스레드를 생성하고 관리하는 복잡함을 줄여준다.

### 📚 Executor 프레임워크의 주요 구성요소
1. Executor 인터페이스
    - 가장 단순한 작업 실행 인터페이스로, execute(Runnable command) 메서드 하나를 가지고 있다.

2. ExecutorService 인터페이스 - 주요 메서드
    - Executor 인터페이스를 확장해서 작업 제출과 제어 기능을 추가로 제공한다.
    - 주요 메서드로는 submit(), close()가 있다.
    - Executor 프레임워크를 사용할 때는 대부분 이 인터페이스를 사용한다.
- ExecutorService의 기본 구현체는 ThreadPoolExecutor이다.

---

### 📚 ExecutorService 코드로 시작하기
```java
ExecutorService es = new ThreadPoolExecutor(2, 2, 0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());
```
- ThreadPoolExecutor는 크게 2가지 요소로 구성되어 있다.
    - 스레드 풀: 스레드를 관리한다.
    - BlockingQueue: 작업을 보관한다. 생산자 소비자 문제를 해결하기 위해 단순한 큐가 아니라, BlockingQueue를 사용한다.

> 🔍생산자
> - es.execute(작업)을 호출하면 내부에서 BlockingQueue에 작업을 보관한다. main 스레드가 생산자가 된다.

> 🔍소비자
> - 스레드 풀에 있는 스레드가 소비자이다. 이후에 소비자 중 하나가 BlockingQueue에 들어있는 작업을 받아서 처리한다.

---

### 📚 ThreadPoolExecutor 생성자
- `corePoolSize`: 스레드 풀에서 관리되는 기본 스레드의 수
- `maximumPoolSize`: 스레드 풀에서 관리되는 최대 스레드의 수
- `keepAliveTime`, `TimeUnit unit`: 기본 스레드 수를 초과해서 만들어진 스레드가 생존할 수 있는 대기 시간이다.
    - 이 시간 동안 처리할 작업이 없다면 초과 스레드는 제거된다.
- `BlockingQueue workQueue`: 작업을 보관할 블로킹 큐.
    - 작업을 보관할 블로킹 큐의 구현체 LinkedBlockingQueue를 사용했는데, 이 블로킹 큐는 작업을 무한대로 저장할 수 있다.

> ThreadPoolExecutor를 생성한 시점에 스레드 풀에 스레드를 미리 만들어두지 않는다.
> - 최초의 작업이 들어오면 이때 작업을 처리하기 위해 스레드를 만든다.
> - 작업이 들어올 때마다 corePoolSize의 크기까지 스레드를 만든다.
    >   - corePoolSize까지 스레드가 생성되고 나면, 이후에는 스레드를 생성하지 않고 앞서 만든 스레드를 재사용한다.

- close()를 호출하면 ThreadPoolExecutor가 종료된다. 이때 스레드 풀에 대기하는 스레드도 함께 제거된다.

> 🔍참고: close() 메서드는 자바 19부터 지원하는 메서드이다. 만약 19 미만 버전을 사용한다면 shutdown()을 호출해라.

---

## ✅ Future 1 시작
- Callable은 다음과 같다.
```java
public interface Callable<V> {
    V call() throws Exception;
}
```
- java.util.concurrent에서 제공되는 기능이다.
- call()의 반환 타입 제네릭 V이다. 따라서 값을 반환할 수 있다.
- throws Exception 예외가 선언되어 있다. 따라서 해당 메서드를 구현하는 모든 메서드는 체크 예외인 Exception과 그 하위 예외를 던질 수 있다.

> 🔍 java.util.concurrent.Executors가 제공하는 newFixedThreadPool(size)를 사용하면 편리하게 ExecutorService를 생성할 수 있다.
>- ```java
>  ExecutorService es = new ThreadPoolExecutor(1, 1, 0,
>                       TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>()); // 기존 코드
>  ExecutorService es = Executors.newFixedThreadPool(1); // 편의 코드
>  ``` 

---

#### 🔍 submit()
```java
<T> Future<T> submit(Callable<T> task); // 인터페이스 정의
```
- ExecutorService가 제공하는 submit()을 통해 Callable 작업으로 전달할 수 있다.
```java
Future<Integer> future = es.submit(new MyCallable()); 
```
- MyCallable의 인스턴스가 블로킹 큐에 전달되고, 스레드 풀의 스레드 중 하나가 이 작업을 실행할 것이다.
- 이때 작업의 처리 결과는 직접 반환되는 것이 아니라 Future라는 특별한 인터페이스를 통해 반환된다.

```java
Integer result = future.get();
```
- future.get()을 호출하면 MyCallable의 call()이 반환한 결과를 받을 수 있다.

---

## 📚 Executor 프레임워크의 장점
- 요청 스레드 결과를 받아야 하는 상황이라면, Callable을 사용한 방식은 Runnable을 사용하는 방식보다 훨씬 편리하다.
- 이 과정에서 내가 스레드를 생성하거나, join()으로 스레드를 제어하거나 한 코드는 전혀 없다. 심지어 Thread라는 코드도 없다.
- 단순하게 ExecutorService에 필요한 작업을 요청하고 결과를 받아서 쓰면 된다.

---

## Future 2 - 분석
- Future는 미래의 결과를 받을 수 있는 객체라는 뜻이다.
    - 전달한 작업의 미래. 이 객체를 통해 전달한 작업의 미래 결과를 받을 수 있다.
- Future는 내부에 작업의 완료와 결과 값을 가진다.
- 생성한 Future를 즉시 반환하기 때문에 요청 스레드는 대기하지 않고, 자유롭게 본인의 다음 코드를 호출할 수 있다.
- future.get()을 하면 작업이 완료되었으면 Future 내부도 작업 완료가 되고 결과 값을 받아 반환할 수 있고, 작업이 완료되지 않았다면 Future 내부도 완료 상태가 아니기에 요청 스레드는 WAITING이 된다.

> 🔍 future.get()을 호출했을 때
> 1. **Future가 완료 상태**
     >    - Future가 완료 상태면 Future에 결과도 포함되어 있다. 이 경우 요청 스레드는 대기하지 않고, 값을 즉시 반환받을 수 있다.
> 2. **Future가 완료 상태가 아님**
     >   - MyCallable 작업이 아직 수행되지 않았거나 또는 수행 중이라는 뜻이다.
           >     - 이때는 어쩔 수 없이 요청 스레드가 결과를 받기 위해 대기해야 한다.
>   - 이처럼 스레드가 어떤 결과를 얻기 위해 대기하는 것을 블로킹(Blocking)이라고 한다.

> 🔍 참고: 블로킹 메서드()
> - Thread.join(), Future.get()과 같은 메서드는 스레드가 작업을 바로 수행하지 않고, 다른 작업이 완료될 때까지 기다리게 하는 메서드이다.
> - 이러한 메서드를 호출하면 호출한 스레드는 지정된 작업이 완료될 때까지 블록(대기)되어 다른 작업을 수행할 수 없다.

---

### 📚 Future를 사용하지 않는 경우
```java
// 잘못 활용한 예 1
Integer result = es.submit(new MyCallable()).get();

// 잘못 활용한 예 2
Future<Integer> future1 = es.submit(task1); // non-blocking
Integer result1 = future1.get(); // blocking -> future2 선언보다 밑에 있어야 함.

Future<Integer> future2 = es.submit(task2); // non-blocking
// non-blocking을 동시에 요청하고 blocking해야 두 개의 작업을 동시에 수행 가능.
```
- Future 없이 결과를 직접 반환을 하게 되면 작업을 반환할 때마다 완료를 기다려야 하므로 두 작업을 호출한다면 task1의 결과를 기다린 다음에 task2를 요청하게 된다.
    - 따라서 작업 하나 당 2초라 가정하면 두 작업을 완료하는데 4초의 시간이 걸리게 되며, 마치 단일 스레드가 작업을 한 것과 비슷한 결과가 나온다.

> 🔍 Future를 반환
> - Future를 반환하면 요청 스레드는 task1, task2를 동시에 요청할 수 있다.
    >   - 따라서 두 작업은 동시에 수행한다. 결과적으로 두 작업을 완료하는데 2초의 시간이 걸린다.

---

## 📌 Future - 정리
### 📚 주요 메서드
1. boolean cancel(boolean mayInterruptIfRunning)
    - 기능: 아직 완료되지 않은 작업을 취소한다.
    - 매개변수: mayInterruptIfRunning
        - cancel(ture): Future를 취소 상태로 변경한다. 단 이미 실행 중이라면 Thread.interrupt()를 호출해서 작업을 중단한다.
        - cancel(false): Future를 취소 상태로 변경한다. 단 이미 실행 중인 작업은 종료하지 않는다.
    - 반환 값: 작업이 성공적으로 취소된 경우 true. 이미 완료되었거나 취소할 수 없는 경우 false.
    - 참고: 취소 상태의 future에 Future.get()을 호출하면 CancellationExcept 런타임 예외가 발생한다.

2. boolean isCanceled()
    - 기능: 작업이 취소되었는지 여부를 확인한다.

3. boolean isDone()
    - 기능: 작업이 완료되었는지 여부를 확인한다.

4. State state()
    - 기능: Future의 상태를 반환한다. 자바 19부터 지원.

5. V get()
    - 기능: 작업이 완료될 때까지 대기하고, 완료되면 결과를 반환한다.
    - 예외
        - InterruptedException: 대기 중에 현재 스레드가 인터럽트 된 경우에 발생.
        - ExecutionException: 작업 계산 중에 예외가 발생한 경우 발생.

6. V get(long timeout, Timeunit unit)
    - 기능: get()과 같은데, 시간 초과되면 예외를 발생.
    - 예외: TimeoutException: 주어진 시간 내에 작업이 완료되지 않는 경우에 발생.
        - get()과 같다.

> 🔍
> - Future.get()은 작업의 결과를 받을 수도 있고, 예외를 받을 수도 있다. 마치 싱글 스레드 상황에서 일반적인 메서드를 호출하는 것 같다.
    >   - Executor 프레임워크가 잘 설계되어 있다는 것이다.
> - getCause()를 호출하면 작업에서 발생한 원본 예외를 받을 수 있다.

---

### 📚 ExecutorService - 작업 컬렉션 처리
- 여러 작업은 한 번에 편리하게 처리하는 invokeAll(), invokeAny() 기능을 제공한다.

1. **invokeAll()**
    ```java
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptException
    ```
    - 모든 Callable 작업을 제출하고, 모든 작업이 완료될 때까지 기다린다.
    - 매개 변수에 long timeout, TimeUnit unit이 있으면, 지정된 시간 내에 모든 Callable 작업을 제출하고 완료될 때까지 기다린다.

2. **invokeAny()**
    ```java
   <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionExcetpion
    ```
   - 하나의 Callable 작업이 완료될 때까지 기다리고, 가장 먼저 완료된 작업의 결과를 반환한다.
   - 완료되지 않은 나머지 작업을 취소한다.
   - 매개 변수 시간 넣을 시 마찬가지로 지정된 시간 내에 하나의 작업이 완료될 때까지 기다리고, 가장 먼저 완료된 결과를 반환 후, 나머지 작업을 취소한다.

---