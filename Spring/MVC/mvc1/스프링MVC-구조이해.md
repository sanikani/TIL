# 4. 스프링 MVC - 구조 이해

### 프론트 컨트롤러 패턴 소개

**프론트 컨트롤러 도입 전**

![Untitled](/assets/mvc1-16.png)

**프론트 컨트롤러 도입 후**

![Untitled](/assets/mvc1-17.png)

**FrontController 패턴 특징**

- 프론트 컨트롤러 서블릿 하나로 클라이언트 요청 받음
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨
- 스프링 웹 MVC의 DispatcheServlet이 FrontController 패턴으로 구현되어 있음

**어댑터 패턴 특징**

- 호환되지 않는 인터페이스를 가진 객체들이 협업할 수 있도록 해주는 구조적 디자인 패턴
- 이미 구축되어 있는 것을 새로운 어떤것에 사용할 때 양 쪽 간의 호환성을 유지해 주기 위해 사용

<aside>
💡 어댑터가 Legacy 인터페이스를 감싸서 새로운 인터페이스로 변환하기 때문에 Wrapper 패턴이라고도 불린다.

</aside>

- 어댑터 패턴을 사용하면 핸들러의 반환값을 어댑터에서 변환시켜 주어 frontController에서 동일하게 처리할 수 있도록 해준다.

![Untitled](/assets/mvc1-18.png)

### DispatcherServlet 구조 살펴보기

- 스프링 MVC도 프론트 컨트롤러 패턴으로 구현되어 있다.
- 스프링 MVC의 프론트 컨트롤러가 디스패처 서블릿이다.

### DispatcherServlet 서블릿 등록

- DispatcherServlet도 부모 클래스에서 HttpServlet을 상속받아서 사용하고, 서블릿으로 동작한다.
    - DispatcherServlet → FramworkServlet → HttpServletBean → HttpServlet
- 스프링 부트는 DispatcherServlet을 서블릿으로 자동으로 등록하면서 모든 경로(urlPatterns=”/”)에 대해서 매핑한다.
- 서블릿이 호출되면 HttpServlet이 제공하는 service()가 호출된다.
- 스프링 MVC는 DispatcherServlet의 부모인 FramworkServlet에서 service()를 오버라이드 해두었다.
- FramworkServlet.service()를 시작으로 여러 메서드가 호출되면서 DispatcherServlet.doDispatch()가 호출된다.

**동작 순서**

1. 핸들러 조회: 핸들러 매핑을 통해 URL에 매핑된 핸들러를 조회한다.
2. 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다.
4. 핸들러 실행: 핸들러 어댑터가 실제 핸들러를 실행한다.
5. ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
6. viewResolver 호출: 뷰 리졸버를 찾고 실행한다.
    - JSP의 경우: InternalResourceViewResolver가 자동 등록되고, 사용된다
7. View 반환: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
    - JSP의 경우 InternalResourceView(JstlView)를 반환하는데, 내부에 forward()로직이 있다.
8. 뷰 렌더링: 뷰를 통해서 뷰를 렌더링 한다.

컨트롤러가 호출되려면 다음 2가지가 필요하다

- **HandlerMapping**
    - 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다.
    - 예) 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.
- **HandlerAdapter**
    - 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
    - 예)Controller 인터페이스를 실행할 수 있는 핸들러 어댑터를 찾고 실행해야 한다.

스프링은 이미 필요한 핸들러 매핑과 핸들러 어댑터를 대부분 구현해두었다. 개발자가 직접 핸들러 매핑과 핸들러 어댑터를 만드는 일은 거의 없다.

![Untitled](/assets/mvc1-19.png)

핸들러 매핑과 핸들러 어댑터 모두 순서대로 찾고 만약 없으면 다음 순서로 넘어간다.

1. **핸들러 매핑으로 핸들러 조회**
    1. HandlerMapping을 순서대로 실행해서 핸들러를 찾는다.
2. **핸들러 어댑터 조회**
    1. HandlerAdapter의 supports()를 순서대로 호출한다.
3. **핸들러 어댑터 실행**
    1. 디스패처 서블릿이 조회한 HandlerAdapter를 실행하면서 핸들러 정보도 함께 넘겨준다.
    2. 핸들러 어댑터는 핸들러를 내부에서 실행하고, 그 결과를 반환한다.

**@RequestMapping**

가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터는 RequestMappingHandlerMapping, RequestMappingHandlerAdapter이다. 지금 스프링에서 주로 사용하는 애노테이션 기반의 컨트롤러를 지원하는 매핑과 어댑터이다.

<aside>
💡 스프링 부트 3.0부터는 클래스 레벨에 @RequestMapping이 있어도 스프링 컨트롤러로 인식하지 않는다. 오직 @Controller가 있어야 스프링 컨트롤러로 인식한다. 참고로 @RestController는 해당 애노테이션 내부에 @Controller를 포함하고 있으므로 인식된다.

</aside>

클래스 레벨에 다음과 같이 @RequestMapping을 두면 메서드 레벨과 조합이 된다

```java
@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2{}
```

### 스프링 MVC - 실용적인 방식

**Model 파라미터**

Model을 파라미터로 받을 수 있다.

**ViewName 직접 반환**

뷰의 논리 이름을 반환할 수 있다.

**@RequestParam 사용**

스프링은 HTTP 요청 파라미터를 @RequestParam으로 받을 수 있다. @ReqeustParam(”username”)은 request.getParameter(”username”)와 거의 같은 코드라 생각하면 된다. GET 쿼리 파라미터, POST Form 방식을 모두 지원한다.

**@RequestMapping → @GetMapping, @PostMapping**

HTTP Method도 함께 구분할 수 있다.