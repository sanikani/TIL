# 5. 스프링 MVC - 기본 기능

**로깅 라이브러리**

스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리가 함께 포함된다.

스프링 부트 로깅 라이브러리는 다음 로깅 라이브러리를 사용한다

- SLF4J
- Logback

SLF4J는 인터페이스이고, 그 구현체로 Logback 같은 로그 라이브러리를 선택하면 된다. 실무에서는 스프링 부트가 기본으로 제공하는 Logback을 대부분 사용한다.

**로그 선언**

```java
private Logger log = LoggerFactory.getLogger(getClass());
private static final Logger log = LoggerFacotry.getLogger(Xxx.class)
@Slf4j //롬복 사용 가능
```

로그 레벨: TRACE > DEBUG > INFO > WARN > ERROR

- 개발 서버는 debug 출력
- 운영 서버는 info 출력

**HTTP 메서드**

@RequestMapping에 method 속성으로 HTTP 메서드를 지정하지 않으면 HTTP 메서드와 무관하게 호출된다.

HTTP 메서드를 축약한 애노테이션을 사용하는 것이 더 직관적이다.

**PathVariable 사용**

```java
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("uesrId") String data){
		log.info("mappingPath userId={}", data);
		return "ok";
}
```

최근 HTTP API는 다음과 같이 리소스 경로에 식별자를 넣는 스타일을 선호한다

- /mapping/userA
- /users/1

- @RequestMapping은 URL 경로를 템플릿화 할 수 있는데, @PathVariable을 사용하면 매칭되는 부분을 편리하게 조회할 수 있다
- @PathVariable의 이름과 파라미터 이름이 같으면 생략할 수 있다.

**미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume**

```java
@PostMapping(value = "/mapping-consume", consumes = "application/json")
public String mappingConsumes(){
		log.info("mappingConsumes");
		return "ok";
```

HTTP 요청의 Content-Type 헤더를 기반으로 미디어 타입으로 매핑한다. 만약 맞지 않으면 HTTP 415 상태코드를 반환한다.

예시) consumes

```java
consumes = "text/plain"
consumes = {"text/plain", "application/*"}
consumes = MediaType.TEXT_PLAIN_VALUE
```

**미디어 타입 조건 매핑 - HTTP 요청 Accept, produce**

```java
@PostMapping(value = "/mapping-produce", prodeces = "text/html")
public String mappingProduces(){
		log.info("mappingProduces");
		return "ok";
```

HTTP 요청의 Accept 헤더를 기반으로 미디어 타입으로 매핑한다.

만약 맞지 않으면 HTTP 406 상태코드를 반환한다.

### HTTP 요청 파라미터 - @ReauestParam

스프링이 제공하는 @RequestParam을 사용하면 요청 파라미터를 매우 편리하게 사용할 수 있다.

- @RequsetParam: 파라미터 이름으로 바인딩
- @ResponseBody: View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
- HTTP 파라미터 이름과 변수 이름이 같으면 @RequestParam(name=”xx”) 생략 가능
- String, int 등의 단순 타입이면 @RequestParam도 생략 가능
- @RequestParam 애노테이션을 생략하면 스프링 MVC는 내부에서 required=false를 적용한다.
- @RequestParm.repuired
    - 파라미터 필수 여부
    - 기본값이 파라미터 필수이다.
- /request-param?username=
파라미터 이름만 있고 값이 없는 경우 → 빈문자로 통과
- 기본형(primitive)에 null
    - /request-param 요청
    - @RequestParam(required = false) int age
        
        null을 int에 입력하는 것은 불가능
        
        따라서 null을 받을 수 있는 Integer로 변경하거나, defaultValue 사용
        
- 파라미터에 값이 없는 경우 defaultValue를 사용하면 기본 값을 적용할 수 있다.
이미 기본 값이 있기 때문에 required는 의미가 없다.
- defaultValue는 빈 문자의 경우에도 설정한 기본 값이 적용된다.

**파라미터를 Map으로 조회하기 - requestParamMap**

```java
@ResponseBody
@RequestMapping("request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap){
}
```

파라미터의 값이 1개가 확실하다면 Map을 사용해도 되지만, 그렇지 않다면 MultiValueMap을 사용

### HTTP 요청 파라미터 - @ModelAttribute

실제 개발을 하면 요청 파라미터를 받아서 필요한 객체를 만들고 그 객체에 값을 넣어주어야 한다. 보통 다음과 같이 코드를 작성한다.

```java
@RequestParam String username;
@ReqeustParam int age;

HelloData data = new HelloData();
data.setUsername(username);
data.setAge(age);
```

스프링은 이 과정을 완전히 자동화해주는 @ModelAttribute 기능을 제공한다.

스프링 MVC는 @ModelAttribute가 있으면 다음을 실행한다

- 대상 객체를 생성한다.
- 요청 파라미터의 이름으로 대상 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩)한다.
- 예) 파라미터의 이름이 username이면 setUsername() 메서드를 찾아서 호출하면서 값을 입력한다.

**프로퍼티**

객체에 getUsername(), setUsername() 메서드가 있으면, 이 객체는 username이라는 프로퍼티를 가지고 있다. username 프로터피의 값을 변경하면 setUsername()이 호출되고, 조회하면 getUsername()이 호출된다.

스프링은 @RequestParam과 @ModelAttribute 생략시 다음과 같은 규칙을 적용한다

- String, int, Integer 같은 단순 타입 = @RequestParam
- 나머지 = @ModelAttribute(argument resolver로 지정해둔 타입 외)

### HTTP 요청 메시지 - 단순 텍스트

요청 파라미터와 다르게, HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 @RequestParam, @ModelAttribute를 사용할 수 없다. (물론 HTML Form 형식으로 전달되는 경우는 요청 파라미터로 인정된다.)

- HTTP 메시지 바디의 데이터를 InputStream을 사용해서 직접 읽을 수 있다.

스프링 MVC는 다음 파라미터를 지원한다.

- InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
- OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력
- HttpEntity: HTTP header, body 정보를 편리하게 조회
    - 메시지 정보를 직접 조회
    - 요청 파라미터를 조회하는 기능과 관계 없음
- HttpEntity는 응답에도 사용
    - 메시지 바디 정보 직접 반환
    - 헤더 정보 포함 가능
    - view 조회X
    
    ```java
    @PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestbodyStringV3(HttpEntity<String> httpEntity){
    		String messageBody = httpEntity.getBody();
    		
    		return new HttpEntity<>("ok");
    }
    ```
    
    HttpEntity를 상속받은 다음 객체들도 같은 기능을 제공한다
    
    - RequestEntity
        - HttpMethod, url 정보가 추가, 요청에서 사용
    - ResponseEntity
        - HTTP 상태 코드 설정 가능, 응답에서 사용
        - return new ResponseEntity<String>(”Hello World”, responseHeaders, HttpStatus.CREATED)

**@RequestBody**

@RequestBody를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 참고로 헤더 정보가 필요하다면 HttpEntity를 사용하거나 @RequestHeader를 사용하면 된다. 

이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 @ReqeustParam, @ModelAttribute와는 전혀 관계가 없다.

**@ResponseBody**

@ResponseBody를 사용하면 응답 결과를  HTTP 메시지 바디에 직접 담아서 전달할 수 있다. 물론 이 경우에도  view를 사용하지 않는다.

### HTTP 요청 메시지 - JSON

```java
@ResponseBody
@PostMapping("/reqeust-body-json-v2")
public String requestBodyJson(@RequestBody String messageBody) throws IOException{
		HelloData data = objectMapper.readValue(messageBody, HelloData.class);
		log.info("username={}, age={}", data.getUsername(), data.getAge());
		return "ok";
}
```

- @RequestBody를 사용해서 HTTP 메시지에서 데이터를 꺼내고 messageBody에 저장한다.
- 문자로 된 JSON 데이터를 Jackson 라이브러리인 objectMapper를 사용해서 자바 객체로 변환한다.

**문자로 변환하고 다시 json으로 변환하는 과정이 불편하다. @ModelAttribute처럼 한번에 객체로 변환하는 방법을 알아보자**

```java
@ResponseBody
@PostMapping("/reqeust-body-json-v3")
public String requestBodyJson(@RequestBody HelloData data){
		log.info("username={}, age={}", data.getUsername(), data.getAge());
		return "ok";
}
```

**@RequestBody 객체 파라미터**

- @RequestBody에 직접 만든 객체를 저장할 수 있다.

HttpEntity나 @RequestBody를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.

HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환시켜준다.

**@RequestBody는 생략 불가능**

@RequestBody를 생략하면 @ModelAttribute가 적용되어버린다.

> HTTP 요청시에 content-type이 application/json인지 확인해야 한다. 그래야 JSON을 처리할 수 있는 HTTP 메시지 컨버터가 실행된다.
> 

응답의 경우에도 @ResponseBody를 사용하면 해당 객체를 HTTP 메시지 바디에 직접 넣어줄 수 있다.

- @RequestBody 요청
    - JSON 요청 → HTTP 메시지 컨버터 → 객체
- @ResponseBody 응답
    - 객체 → HTTP  메시지 컨버터 → JSON 응답

### HTTP 응답 - 정적 리소스, 뷰 템플릿

스프링에서 응답 데이터를 만드는 방법은 크게 3가지이다

- 정적 리소스
- 뷰 템플릿 사용
- HTTP 메시지 사용

**@RestController**

@Controller 대신에 @RestController애노테이션을 사용하면 해당 컨트롤러에 모두 @ResponseBody가 적용되는 효과가 있다. 따라서 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 데이터를 입력한다. 이름 그래도 Rest API를 만들 때 사용하는 컨트롤러이다.

### HTTP 메시지 컨버터

뷰 템플릿으로  HTML을 생성해서 응답하는 것이 아니라, HTTP API처럼 JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는 경우 HTTP 메시지 컨버터를 사용하면 편리하다.

**@ResponseBody 사용 원리**

- @ResponseBody를 사용
    - HTTP의 Body에 문자 내용을 직접 반환
    - viewResolver 대신에 HttpMessageConverter가 동작
    - 기본 문자 처리: StringHttpMessageConverter
    - 기본 객체 처리: MappingJackson2HttpMessageConverter
    - byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음

**스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다.**

- HTTP 요청: @RequestBody, HttpEntity(RequestEntity)
- HTTP 응답: @ResponseBody, HttpEntity(ResponseEntity)

HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다

- canRead(), canWrite(): 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
- read(), write(): 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기

스프링 부트는 다양한 메시지 컨버터를 제공하는데, 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정한다. 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.

**HTTP 요청 데이터 읽기**

- HTTP 요청이 오고, 컨트롤러에서 @RequestBody, HttpEntity 파라미터를 사용한다.
- 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead()를 호출한다.
    - 대상 클래스 타입을 지원하는가.
        - 예) @RequestBody의 대상 클래스(byte[], String, HelloData)
    - HTTP 요청의 Content-Type 미디어 타입을 지원하는가
        - 예) text/plane, application/json, */*
    - canRead() 조건을 만족하면 read()를 호출해서 객체 생성하고, 반환한다.

**HTTP 응답 데이터 생성**

- 컨트롤러에서 @ResponseBody, HttpEntity로 값이 반환된다.
- 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 canWrite()를 호출한다.
    - 대상 클래스 타입을 지원하는가
        - 예) return의 대상 클래스(byte[], String, HelloData)
    - HTTP 요청의 Accept 미디어 타입을 지원하는가.(더 정확히는 @RequestMapping의 produeces)
        - 예) text/plane, application/json, */*
    - canWrite() 조건을 만족하면 write()를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다

### 요청 매핑 핸들러 어댑터 구조

HTTP 메시지 컨버터는 스프링 MVC 어디에서 사용되는 것일까?

@RequestMapping을 처리하는 핸들러 어댑터인 RequestMappingHandlerAdapter(요청 매핑핸들러 어댑터)를 살펴보자

**ReqpuestMappingHandlerAdapter 동작 방식**

![Untitled](/assets/mvc1-20.png)

**ArgumentResolver**

애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있다.

HttpServletRequest, Model, @RequestParam, @ModelAttribute 같은 애노테이션, 그리고 @ResponseBody, HttpEntity 같은 HTTP 메시지를 처리하는 부분까지 매우 큰 유연함을 보인다. 이렇게 파라미터를 유연하게 처리할 수 있는 이유가 바로 ArgumentResolver 덕분이다.

애노테이션 기반 컨트롤러를 처리하는 RequestMappingHandlerAdapter는 ArgumentResolver를 호출해서 컨트롤러가 필요로 하는 다양한 파라미터의 값을 생성한다. 그리고 파라미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.

```java
public interface HandlerMethodArgumentResolver {

	boolean supportsParameter(MethodParameter parameter);

	@Nullable
	Object resolveArgument(
	MethodParameter parameter, @Nullable
	ModelAndViewContainer mavContainer,
	NativeWebRequest webRequest, 
	@Nullable WebDataBinderFactory binderFactory) throws Exception;
}
```

**동작방식**

ArgumentResolver의 supportsParameter()를 호출해서 해당 파라미터를 지원하는지 체크하고, 지원하면 resolveArgument()를 호출해서 실제 객체를 생성한다. 그리고 이렇게 생성된 객체가 컨트롤러 호출시 넘어간다.

**ReturnValueHandler**

응답 값을 변환하고 처리한다. 컨트롤러에서 String으로 뷰 이름을 반환해도 동작하는 이유가 바로 ReturnValueHandler 덕분이다.

### HTTP 메시지 컨버터

**HTTP 메시지 컨버터 위치**

![Untitled](/assets/mvc1-21.png)

HTTP 메시지 컨버터를 사용하는 @RequestBody도 컨트롤러가 필요로 하는 파라미터의 값에 사용된다. @ResponseBody의 경우도 컨트롤러의 반환 값을 이용한다.

요청의 경우 @RequestBody를 처리하는 ArgumentResolver가 있고, HttpEntity를 처리하는 ArgumentResolver가 있다. 이 ArgumentResolver들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성하는 것이다.

응답의 경우 @ResponseBody와 HttpEntity를 처리하는 ReturnValueHandler가 있다. 그리고 여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.

스프링 MVC는 @RequestBody, @ResponseBody가 있으면 

RequestResponseBodyMethodProcessor (ArgumentResolver)
HttpEntity 가 있으면 HttpEntityMethodProcessor (ArgumentResolver)를 사용한다