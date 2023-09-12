# 3. 서블릿, JSP, MVC 패턴

## MVC 패턴 - 개요

### 너무 많은 역할

하나의 서블릿이나 JSP만으로 비즈니스 로직과 뷰 렌더링까지 모두 처리하게 되면, 너무 많은 역할을 하게된다.

### 변경의 라이프 사이클

예를 들어 UI를 일부 수정하는 일과 비즈니스 로직을 수정하는 일은 각각 다르게 발생할 가능성이 매우 높고 대부분 서로에게 영향을 주지 않는다. 이렇게 변경의 라이프 사이클이 다른 부분을 하나의 코드로 관리하는 것은 유지보수하기 좋지 않다.

### 기능 특화

특히 JSP 같은 뷰 템플릿은 화면을 렌더링 하는데 최적화 되어 있기 때문에 이 부분의 업무만 담당하는 것이 가장 효과적이다.

### Model View Controller

MVC 패턴은 하나의 서블릿이나 JSP로 처리하던 것을 컨트롤러와 뷰라는 영역으로 서로 역할을 나눈 것을 말한다.

- 컨트롤러: HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다. 그리고 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다.
- 모델: 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고, 화면을 렌더링하는 일에 집중할 수 있게 된다.
- 뷰: 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다. 여기서는 HTML을 생성하는 부분을 말한다.

> 컨트롤러에 비즈니스 로직을 둘 수도 있지만, 이렇게 되면 컨트롤러가 너무 많은 역할을 담당한다. 그래서 일반적으로 비즈니스 로직은 서비스라는 계층을 별도로 만들어서 처리한다. 그리고 컨트롤러는 비즈니스 로직이 있는 서비스를 호출하는 역할을 담당한다. 참고로 비즈니스 로직을 변경하면 비즈니스 로직을 호출하는 컨트롤러의 코드도 변경될 수 있다.
> 

![Untitled](/assets/mvc1-14.png)

**MVC 패턴 1**

![Untitled](/assets/mvc1-15.png)

![Untitled](/assets/mvc패턴2.png)

> **redirect vs foward**
리다이렉트는 실제 클라이언트에 응답이 나갔다가, 클라이언트가 redirect 경로로 다시 요청한다. 따라서 클라이언트가 인지할 수 있고, URL 경로도 실제로 변경된다. 반면에 포워드는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 전혀 인지하지 못한다.
> 

```html
<form action="save" method="post">
 username: <input type="text" name="username" />
 age: <input type="text" name="age" />
 <button type="submit">전송</button>
</form>
```

위와 같이 action에 절대 경로가 아니라 상대 경로를 사용하면 현재 URL에 속한 계층 경로 + save가 호출된다.

Servlet은 HttpServletRequest를 Model로 사용한다. request가 제공하는 setAttribute()를 사용하면 request 객체에 데이터를 보관해서 뷰에 전달할 수 있다. 뷰는 request.getAttribute()를 사용해서 데이터를 꺼내면 된다.

### MVC 패턴 - 한계

컨트롤러에 중복이 많고, 필요하지 않는 코드들도 많이 보인다.

**MVC 컨트롤러의 단점**

**포워드 중복**

View로 이동하는 코드가 항상 중복 호출되어야 한다.

```java
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```

**ViewPath에 중복**

```java
String viewPath = "/WEB-INF/views/new-form.jsp";
```

**사용하지 않는 코드**

다음 코드를 사용할 때도 있고, 사용하지 않을 때도 있다.

```java
HttpServletRequest request, HttpServletResponse response
```

그리고 이런 HttpServletRequest, HttpServletResponse를 사용하는 코드는 테스트 케이스를 작성하기도 어렵다.

공통 처리가 어렵다는 문제가 있다. 이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다. 프론트 컨트롤러 패턴을 도입하면 이런 문제를 해결할 수 있다.