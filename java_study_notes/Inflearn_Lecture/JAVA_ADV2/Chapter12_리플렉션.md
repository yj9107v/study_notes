# 📚 Chapter 12: 리플렉션

> 📌 공부 날짜: 2025/09/01
> - `JAVA `- 고급 2편
> - `References`: [인프런 - 김영한의 실전 자바](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EA%B3%A0%EA%B8%89-2)

---

## ✅ 리플렉션이 필요한 이유
- 우리가 앞서 커맨드 패턴으로 만든 서블릿은 아주 유용하지만, 몇 가지 단점이 있다.
  - 문제 1: 하나의 클래스에 하나의 기능만 만들 수 있다.
  - 문제 2: 새로 만든 클래스를 URL 경로와 항상 매핑해야 한다.

> 자바 프로그램 실행 중에 이름으로 메서드를 찾고, 또 찾은 메서드를 호출하려면 자바의 리플렉션 기능을 먼저 알아야 한다.

---

## ✅ 클래스와 메타데이터
- 클래스가 제공하는 다양한 정보를 동적으로 분석하고 사용하는 기능을 **리플렉션(Reflection)**이라 한다.
  - 리플렉션을 통해 프로그램 실행 중에 클래스, 메서드, 필드 등에 대한 정보를 얻거나, 새로운 객체를 생성하고 메서드를 호출하며, 필드의 값을 읽고 쓸 수 있다.

리플렉션을 통해 얻을 수 있는 정보는 다음과 같다.
- **클래스의 메타데이터**: 클래스 이름, 접근 제어자, 부모 클래스, 구현된 인터페이스 등.
- **필드 정보**: 필드의 이름, 타입, 접근 제어자를 확인하고, 해당 필드의 값을 읽거나 수정할 수 있다.
- **메서드 정보**: 메서드 이름, 반환 타입, 매개변수 정보를 확인하고, 실행 중에 동적으로 메서드를 호출할 수 있다.
- **생성자 정보**: 생성자의 매개변수 타입과 개수를 확인하고, 동적으로 객체를 생성할 수 있다.

---

### 📚 클래스 메타데이터 조회
- 클래스의 메타데이터는 `Class`라는 클래스로 표현된다. 그리고 `Class`라는 클래스를 획득하는 3가지 방법이 있다.

#### 1. 클래스에서 찾기
```java
Class<BasicData> basicDataClass1 = BasicData.class;
```
- 클래스명에 `.class`를 사용하면 획득할 수 있다.

#### 2. 인스턴스에서 찾기
```java
BasicData helloInstance = new BasicData();
Class<? extends BasicData> basicDataClass2 = helloInstance.getClass();
```
- 인스턴스에서 `.getClass()` 메서드를 호출하면 획득할 수 있다.
- 반환 타입을 보면 `Class<? extends BasicData>`로 표현되는데, 실제 인스턴스가 `BasicData` 타입일 수도 있지만, 그 자식 타입일 수도 있기 때문이다.

#### 3. 문자로 찾기
```java
String className = "reflection.data.BasicData";
Class<?> basicData3 = Class.forName(className);
```
- 이 부분이 가장 흥미로운 부분, 단순히 문자로 클래스의 메타데이터를 조회할 수 있다.
  - 예를 들어서, 콘솔에서 사용자 입력으로 원하는 클래스를 동적으로 찾을 수 있다는 뜻이다.

#### 🔍 기본 정보 탐색
- 이렇게 찾은 클래스 메타데이터로 어떤 일들을 할 수 있는가?
  - 클래스 이름, 패키지, 부모 클래스, 구현한 인터페이스, 수정자 정보 등 다양한 정보를 획득할 수 있다.

참고로 수정자는 접근 제어자와 비 접근 제어자(기타 수정자)로 나눌 수 있다.
- 접근 제어자: `public`, `protected`, `default`, `(package-private)`, `private`
- 비 접근 제어자: `static`, `final`, `abstract`, `synchronized`, `volatile` 등

`getModifiers()`를 통해 수정자가 조합된 숫자를 얻고, `Modifier`를 사용해서 실제 수정자 정보를 확인할 수 있다.

---

## ✅ 메서드 탐색과 동적 호출
- 클래스 메타데이터를 통해 클래스가 제공하는 메서드의 정보를 확인해 보자.
- `Class.getMethods()` 또는 `Class.getDeclaredMethods()`를 호출하면 `Method`라는 메서드의 메타데이터를 얻을 수 있다. 이 클래스는 메서드의 모든 정보를 가지고 있다.

#### 🔍 getMethods() vs getDeclaredMethods()
- `getMethods()`: 해당 클래스와 상위 클래스에서 상속된 모든 public 메서드를 반환
- `getDeclaredMethods()`: **해당 클래스에서 선언된 모든 메서드를 반환하며**, 접근 제어자에 관계없이 반환. 상속된 메서드는 포함하지 않음.

---

### 📚 동적 메서드 호출
- 리플렉션을 사용하면 매우 다양한 체크 예외가 발생한다.

#### 🔍 일반적인 메서드 호출 - 정적
- 인스턴스의 참조를 통해 메서드를 호출하는 방식이 일반적인 메서드 호출 방식이다.
- 이 방식은 코드를 변경하지 않는 이상 `call()` 메서드를 호출했다면, 대신 다른 메서드로 변경하는 것이 불가능하다.
  - `helloInstance.call()`
- 호출하는 메서드가 이미 코드로 작성되어서 **정적**으로 변경할 수 없는 상태이다.

#### 🔍 동적 메서드 호출 - 리플렉션 사용
```java
String methodName = "hello";
Mehtod method1 = helloClass.getMethod(methodName, String.class);
Object returnValue = method1.invoke(helloInstance, "hi");
```
- 클래스 메타데이터가 제공하는 `getMethod()`에 메서드 이름, 사용하는 매개변수의 타입을 전달하면 원하는 메서드를 찾을 수 있다.
- 여기서는 hello라는 이름에 `String` 매개변수가 있는 `hello(String)` 메서드를 찾는다.
- `Method.invoke()` 메서드에 실행할 인스턴스와 인자를 전달하면, 해당 인스턴스에 있는 메서드를 실행할 수 있다.
- 여기서는 `BasicData helloInstance = new BasicData()` 인스턴스에 있는 `hello(String)` 메서드를 호출한다.

여기서 메서드를 찾을 때 `helloClass.getMethod(methodName, String.class)`에서 `methodName` 부분이 String 변수로 되어 있는 것을 확인할 수 있다.
`methodName`은 변수이므로 예를 들어, 사용자 콘솔 입력을 통해서 얼마든지 호출할 `methodName`을 변경할 수 있다.

따라서 여기서 호출할 메서드 대상은 정적으로 딱 코드에 정해진 것이 아니라, 언제든지 동적으로 변경할 수 있다. 그래서 `동적 메서드 호출`이라 한다.

---

## ✅ 필드 탐색과 값 변경
#### 🔍 getFields() vs getDeclaredFields()
- `getFields()`: 해당 클래스와 상위 클래스에서 상속된 모든 public 필드를 반환
- `getDeclaredFields()`: **해당 클래스에서 선언된 모든 필드를 반환하며**, 접근 제어자에 관계없이 반환. 상속된 필드는 포함하지 않음.

### 📚 필드 값 변경
```java
nameField.setAcessible(true);
```
- 리플렉션은 `private` 필드에 접근할 수 있는 특별한 기능을 제공.
- 참고로 `setAcessible(true`기능은 `Method`도 제공한다. 따라서 `private` 메서드를 호출할 수도 있다.

```java
nameField.set(user, "userB");
```
- `user` 인스턴스에 있는 `nameField`의 값을 `userB`로 변경한다.

#### ⚠️ 리플렉션과 주의사항
- 리플렉션을 활용하면 `private` 접근 제어자에도 직접 접근해서 값을 변경할 수 있다.
  - 하지만 이는 객체 지향 프로그래밍의 원칙을 위반하는 행위로 간주될 수 있다.
  - `private` 접근 제어자는 클래스 내부에서만 데이터를 보호하고, 외부에서의 직접적인 접근을 방지하기 위해 사용된다.
  - 리플렉션을 통해 이러한 접근 제한을 무시하는 것은 캡슐화 및 유지보수성에 악영향을 미칠 수 있다.
  - 예를 들어, 클래스의 내부 구조나 구현 세부 사항이 변경될 경우 리플렉션을 사용한 코드는 쉽게 깨질 수 있으며, 이는 예상치 못한 버그를 초래할 수 있다.
- 따라서 리플렉션을 사용할 때는 반드시 신중하게 접근해야 하며, 가능한 경우 접근 메서드(예: getter, setter)를 사용하는 것이 바람직하다.
  - 리플렉션은 주로 테스트나 라이브러리 개발 같은 특별한 상황에서 유용하게 사용되지만, 일반적인 애플리케이션 코드에서는 권장되지 않는다.
  - 이를 무분별하게 사용되면 코드의 가독성과 안전성을 크게 저하시킬 수 있다.

---

## ✅ 리플렉션 - 활용 예제
- 프로젝트에서 데이터를 저장해야 하는데, 저장할 때는 반드시 `null`을 사용하면 안 된다고 가정해 보자.
  - 이 경우 `null` 값을 다른 기본 값으로 모두 변경해야 한다.

이 문제를 해결하려면 각각의 객체에 들어있는 데이터를 직접 다 찾아서 값을 입력해야 한다. 만약 `User`, `Team` 뿐만 아니라 `Order`, `Cart`, `Delivery` 등등 수많은 객체에 해당 기능을 적용해야 한다면 매우 번거로운 코드를 작성해야 한다.

### 📚 리플렉션을 활용한 필드 기본 값 도입
```java
public class FieldUtil {
    
    public static void nullFieldToDefault(Object target) throws IllegalAccessException {
        Class<?> aClass = targer.getClass();
        Field[] declaredFields = aClass.getDeclaredFields();
        for (Field field : declaredFields) {
            field.setAccessible(true);
            if (field.get(target) != null) continue;
            
            if (field.getType() == String.class) {
                field.set(targer, "");
            } else if (field.getType == Integer.class) {
                field.set(target, 0);
            }
        }
    }
}
```
- 리플렉션을 사용한 덕분에 수많은 객체에 매우 편리하게 기본 값을 적용할 수 있게 되었다.
- 이처럼 리플렉션을 활용하면 기존 코드로 해결하기 어려운 공통 문제를 손쉽게 처리할 수도 있다.

---

## ✅ 생성자 탐색과 객체 생성
- 리플렉션을 활용하면 생성자를 탐색하고, 또 탐색한 생성자를 사용해서 객체를 생성할 수 있다.
- `Class.getConstructors()`, `Class.getDeclaredConstructors()`

`Class.forName("reflection.data.BasicData")`을 사용해서 클래스 정보를 동적으로 조회했다.

`getDeclaredConstructor(String.class)`: 생성자를 조회한다.
- 여기서는 매개변수로 `String`을 사용하는 생성자를 조회한다.

`constructor.setAccessible(true)`를 사용해서 private 생성자를 접근 가능하게 만들 수 있다..

```java
Object instance = constructor.newInstance("hello");
```
- 찾은 생성자를 사용해서 객체를 생성한다. 여기서는 "hello"라는 인자를 넘겨준 것.

```java
Method method1 = aClass.getDeclaredMethod("call");
method1.invoke(instance);
```
- 앞서 생성한 인스턴스에 `call`이라는 이름의 메서드를 동적으로 찾아서 호출한다.

클래스를 동적으로 찾아서 인스턴스를 생성하고, 메서드도 동적으로 호출했다. 
코드 어디에도 `BasicData`의 타입이나 `call()` 메서드를 직접 호출하는 부분을 직접 코딩하지 않았다.

클래스를 찾고 생성하는 방법도, 그리고 생성한 클래스의 메서드를 호출하는 방법도 모두 동적으로 처리한 것이다.

#### 🤔 참고
- 스프링 프레임워크나 다른 프레임워크 기술들을 사용해 보면, 내가 만든 클래스를 프레임워크가 대신 생성해 줄 때가 있다.
  - 그때가 되면 리플렉션과 동적 객체 생성 방법들이 떠오를 것이다.

---

## ✅ HTTP 서버 6 - 리플렉션 서블릿 (활용)
- 리플렉션을 활용한 서블릿 기능

```java
public class ReflectController {
    
    public void site1(HttpRequest request, HttpResponse response) {}
  
    public void site2(HttpRequest request, HttpResponse response) {}
  
    public void search(HttpRequest request, HttpResponse response) {}
}
```
- 개발자는 이런 Xxx컨트롤러라는 기능만 개발하면 된다.
- 이러한 컨트롤러의 메서드를 리플렉션으로 읽고 호출할 것이다.

#### 🔍 참고 - 컨트롤러 용어
- 컨트롤러는 애플리케이션의 제어 흐름을 '제어(control)'한다. 요청을 받아 적절한 비즈니스 로직을 호출하고, 그 결과를 뷰에 전달하는 등의 작업을 수행한다.

리플렉션을 활용하면 메서드 이름을 알 수 있다. 예를 들어, `/site`이 입력되면 `site1()`이라는 메서드를 이름으로 찾아서 호출하는 것이다. 이렇게 하면 번거로운 매핑 작업을 제거할 수 있다.

---

### 📚 리플렉션을 처리하는 서블릿 구현
- 서블릿은 자바 진영에서 이미 표준을 사용하는 기술이다. 따라서 서블릿은 그대로 사용하면서 새로운 기능을 구현해 보자.

```java
public class ReflectionServlet implements HttpServlet {
    
    private final List<Object> controllers;
    
    public ReflectionServlet(List<Object> controllers) {
        this.controllers = controllers;
    }
    
    @Override
    public void service(HttpRequest request, HttpResponse response) throws IOException {
        String path = request.getPath();
        
        for (Object controller : controllers) {
            Class<?> aClass = controller.getClass();
            Method[] methods = aClass.getDeclaredMethods();
            for (Method method : methods) {
                String methodName = method.getName();
                if (path.equals("/" + methodName)) {
                    invoke(controller, method, request, response);
                    return;
                }
            }
        }
        throw new PageNotFoundException("request= " + path);
    }
    
    private static void invoke(Object controller, Method method, HttpRequest request, HttpResponse response) {
        try {
            method.invoke(controller, method, request, response);
        } catch (InvocationTargetException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
}
```
- `List<Object> controllers`: 생성자를 통해 여러 컨트롤러들을 보관할 수 있다.

이 서블릿은 요청이 오면 모든 컨트롤러를 순회한다. 그리고 선언된 메서드 정보를 통해 URL의 요청 경로와 메서드 이름이 맞는지 확인한다.
만약 메서드 이름이 맞다면 `invoke`를 통해 해당 메서드를 동적으로 호출한다.

```java
public class ServerMainV6 {
    
    private final int PORT = 12345;

  public static void main(String[] args) throws IOException {
    List<Object> controllers = List.of(new SiteController(), new SearchController());
    HttpServlet reflectionServlet = new ReflectionServlet(controllers);
    
    ServletManager servletManager = new ServletManager();
    servletManager.setDefaultServlet(reflectionServlet);
    servletManager.add("/", new HomeServlet());
    servletManager.add("/", new DiscardServlet());
    
    HttpServer server = new HttpServer(PORT, servletManager);
    server.start();
  }
}
```
#### 🔍 new ReflectionServlet(controllers)
- 리플렉션 서블릿을 생성하고, 사용할 컨트롤러들을 인자로 전달한다.

#### 🔍 servletManager.setDefaultServlet(reflectionServlet)
- 이 부분이 중요하다. 리플렉션 서블릿을 기본 서블릿으로 등록. 이렇게 되면 다른 서블릿에서 경로를 찾지 못할 때 우리가 만든 리플렉션 서블릿 항상 호출된다.
- 그리고 다른 서블릿은 등록하지 않는다. 따라서 항상 리플렉션 서블릿이 호출된다.
- `HomeServlet`과 `/favicon.ico`는 등록해야 한다. `/`와 `favicon.ico`라는 이름으로 메서드를 만들 수 없기 때문이다.

#### 🤔 남은 문제점 (다음 단원)
- 리플렉션 서블릿은 요청 URL과 메서드 이름이 같다면 해당 메서드를 동적으로 호출할 수 있다. 하지만 요청 이름과 메서드 이름을 다르게 하고 싶다면 어떻게 해야 할까?
  - 예를 들어서 `/site`이라고 와도 `page1()`과 같은 다른 이름의 메서드를 호출하고 싶다면 어떻게 해야 할까?
- 앞서 `/`, `favicon.ico`와 같이 자바 메서드 이름으로 처리하기 어려운 URL은 어떻게 해결할 수 있을까?
- URL은 주로 `-`(dash)를 구분자로 사용한다. `/add-member`와 같은 URL은 어떻게 해결할 수 있을까?

---
