# 📚 Chapter 14: HTTP 서버 활용

> 📌 공부 날짜: 2025/09/09
> - `JAVA `- 고급 2편
> - `References`: [인프런 - 김영한의 실전 자바](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EA%B3%A0%EA%B8%89-2)

---

## ✅ HTTP 서버 7 - 애노테이션 서블릿 - 동적 바인딩, 성능 최적화
애노테이션 기반의 컨트롤러와 서블릿을 만들어보자.

- `site1()`, `home()`의 경우 `HttpRequest request`가 전혀 필요하지 않다. 
  - 컨트롤러의 메서드를 만들 때, `HttpRequest request, HttpResponse response` 중에 딱 필요한 메서드만 유연하게 받을 수 있도록 해보자.

2가지 개선할 점
- 성능 최적화
  - O(n) -> O(1)로 변경하기
- 중복 매핑 문제
  - 개발자가 실수로 `@Mapping`에 같은 `/site1()`를 2개 정의하면 어떻게 될까
  - 이 경우 현재 로직에서는 먼저 찾은 메서드가 호출, **개발에서 가장 나쁜 것은 모호한 것이다!**

애노테이션
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface Mapping {
    String value();
}
```

```java
public class AnnotationServlet implements HttpServlet {
    
    private final Map<String, ControllerMethod> pathMap;
    
    public AnnotationServlet(List<Object> controllers) {
        this.pathMap = new HashMap<>();
        initializePathMap(controllers);
    }
    
    private void initializePathMap(List<Object> controllers) {
        for (Object controller : controllers) {
            for (Method method : controller.getClass().getDeclaredMethods()) {
                if (method.isAnnotationPresent(Mapping.class)) {
                    String path = method.getAnnotation(Mapping.class).value();
                    
                    // 중복 경로 체크
                    if (pathMap.containKey(path)) {
                        ControllerMethod controllerMethod = pathMap.get(path);
                        throw new IllegalArgumentException("경로 중복 등록, path= " + path + 
                                ", method= " + method + ", 이미 등록된 메서드= " + controllerMethod);
                    }
                    
                    pathMap.put(path, new ControllerMethod(controller, method));
                }
            }
        }
    }
    
    @Override
    public void service(HttpRequest request, HttpResponse response) throws IOException {
        String path = request.getPath();
        ControllerMethod controllerMethod = pathMap.get(path);
        
        if (controllerMethod == null) {
            throw new PageNotFoundException("request= " + path);
        }
        
        controllerMethod.invoke(request, response);
    }
    
    private static class ControllerMethod {
        private final Object controller;
        private final Method method;

        public ControllerMethod(Object controller, Method method) {
            this.controller = controller;
            this.method = method;
        }

        public void invoke(HttpRequest request, HttpResponse response) {
            Class<?>[] parameterTypes = method.getParameterTypes();
            Object[] args = new Object[parameterTypes.length];

            for (int i = 0; i < parameterTypes.length; i++) {
                if (parameterTypes[i] == HttpRequest.class) {
                    args[i] = request;
                } else if (parameterTypes[i] == HttpResponse.class) {
                    argsp[i] = response;
                } else {
                    throw new IllegalArgumentException("Unsupported parameter type: " + parameterTypes[i]);
                }
            }

            try {
                method.invoke(controller, args);
            } catch (IllegalAccessException | InvocationTargetException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```
- `AnnotationServlet`을 생성하는 시점에 `@Mapping`을 사용하는 컨트롤러의 메서드를 모두 찾아서 `pathMap`에 보관한다.
  - `key= 경로, value = ControllerMethod` 구조이다.
- `ControllerMethod`: `@Mappoing`의 대상 메서드와 메서드가 있는 컨트롤러 객체를 캡슐화했다.

```java
public class Controller {
    @Mapping("/")
    public void home (HttpResponse response) {
        response.writeBody("<h1>home</h1>");
        // response . . .
    }
    
    @Mapping("/site1")
    public void site1(HttpResponse response) {
        response.writeBody("<h1>site1</h1>");
    }
    
    @Mapping("/search")
    public void search(HttpRequest request, HttpResponse response) {
        String query = request.getParameter("q");
        
        response.writeBody("<h1>Search</h1>");
        // response.writeBody() . . .
    }
}
```
- 리플렉션에서 사용한 코드와 비슷하지만, 차이는 호출할 메서드를 찾을 때, 메서드의 이름을 비교하지 않고, 메서드의 `@Mapping` 애노테이션을 찾고, 그곳의 `value` 값으로 비교한다.


```java
public class ServerMain {
    
    private static final int PORT = 12345;

  public static void main(String[] args) throws IOException {
      List<Object> controllers = List.of(new Controller());
      HttpServlet servlet = new AnnotationServlet(controllers);
      
      ServletManager servletManager = new ServletManager();
      servletManager.setDefaultServlet(servlet);
      servletManager.add("/favicon.ico", new DiscardServlet());
      HttpServer server = new HttpServer(PORT, servletManager);
      server.start();
  }
}
```
- 서버를 실행하는 시점에 바로 오류가 발생하고, 서버 실행이 중단된 것을 확인할 수 있다.
  - 이렇게 서버 실행 전에 발견할 수 있는 오류는 아주 좋은 오류이다.

#### 🔍 3가지 오류
- **컴파일 오류**: 가장 좋은 오류. 프로그램 실행 전에 개발자가 가장 빠르게 문제를 확인할 수 있다.
- **런타임 오류 - 시작 오류**: 자바 프로그램이나 서버를 시작하는 시점에 발견할 수 있는 오류이다.
  - 문제를 아주 빠르게 발견할 수 있기 때문에 좋은 오류이다. 고객이 문제를 인지하기 전에 수정하고 해결 가능.
- **런타임 오류 - 작동 오류**: 고객이 특정 기능을 작동할 때 발생하는 오류. 원인 파악과 문제 해결에 가장 많은 시간이 걸리고, 가장 큰 피해를 주는 오류


#### 🔍 정리
- 현대의 웹 프레임워크들은 대부분 애노테이션을 사용해서 편리하게 호출 메서드를 찾을 수 있는 지금과 같은 방식을 제공한다.
- `AnnotationServlet`에서 호출할 컨트롤러의 메서드의 매개변수를 먼저 확인한 다음에 매개변수에 필요한 값을 동적으로 만들어서 전달했다.
  - 덕분에 컨트롤러의 메서드는 자신에게 필요한 값만 선언하고, 전달받을 수 있다.
  - 이런 기능을 확장하면 `HttpRequest`, `HttpResponse` 뿐만 아니라 다양한 객체들도 전달할 수 있다.

---

## 📌 정리
### 구성 역할, 사용 역할
- `ServerMain` 클래스의 역할은 아주 재미있다. 본인이 어떤 코드 블록을 만든 것이 아니라, 지금까지 있는 코드 블록들을 조립하는 일을 한다.
- 이것은 마치 컴포넌트를 구성(Configuration)하는 것 같다.
- `ServerMain`는 프로젝트를 구성하는 역할을 담당한다.
- 나머지 클래스들은 실제 기능을 제공하는 사용 역할을 담당한다.
- 이렇게 구성하는 역할과 사용하는 역할을 명확하게 분리해 두면 다음과 같은 장점이 있다.

#### 🔍 유연성 향상
- 프로젝트 구성을 쉽게 변경할 수 있다. 다른 컨트롤러를 사용하고 싶을 때, `ServerMain`만 수정하면 된다.

---
#### 🔍 테스트 용이성
- 각 컴포넌트를 독립적으로 테스트할 수 있다.
- 구성 로직과 실제 기능 로직이 분리되어 있어 단위 테스트가 더 쉬워진다.

---
#### 🔍 코드 재사용성 증가
- 각 컴포넌트는 독립적이므로 다른 프로젝트에서도 쉽게 재사용할 수 있다.

---
#### 🔍 관심사의 분리
- 구성 로직과 비즈니스 로직이 분리되어 각 부분에 집중할 수 있다.

---
#### 🔍 유지보수 용이성
- 전체 시스템의 구조를 이해하기 쉬워지며, 특정 부분을 수정할 때 다른 부분에 미치는 영향을 최소화할 수 있다.

---
#### 🔍 확장성 개선
- 새로운 기능이나 컴포넌트를 추가할 때, 기존 코드를 쉽게 수정하지 않고도 `ServerMain`에 새로운 구성을 추가할 수 있다.

---

이러한 설계 방식을 효과적으로 구현하려면 소프트웨어 개발의 주요 원칙들을 준수해야 한다.

특히 **다형성, 개방-폐쇄 원칙(OCP), 그리고 의존관계 주입(DI) 원칙이 중요하다.**
이러한 접근법은 대규모 프로젝트에서 특히 유용하다.

하지만 구성과 사용의 역할을 명확히 구분하고, 소프트웨어 개발의 핵심 원칙들을 적용하는 것은 쉽지 않다.
그러나 이러한 과정을 크게 간소화해 주는 도구가 바로 **스프링 프레임워크**이다.

---
