# Spring MVC 이해하기
MVC 1편(2) (김영한)

> 들어가기에 앞서.. 로깅에 대해서 먼저 알고가자!

## 로깅
*로깅 라이브러리*
* SLF4J
* Logback

로그 라이브러리는 Logback, Log4J, Log4J2 등등 수 많은 라이브러리가 있는데,
그것을 통합해서 인터페이스로 제공하는 것이 바로 `SLF4J` 라이브러리 이다.<br>
실무에서는 스프링 부트가 기본으로 제공하는 **Logback을 대부분 사용**한다.

### 로그 선언
```java
private Logger log = LoggerFactory.getLogger(getClass());
```

```java
private static final Logger log = LoggerFactory.getLogger(Xxx.class);
```

```java
@Slf4j
public class className {
    public logTest() {
        log.info("로그!");   
    }
} 
```
#### 참고
* https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging

## HTTP 메서드 축약 애노테이션
* @GetMapping
* @PostMapping
* @PutMapping
* @DeleteMapping
* @PatchMapping

*경로 변수(다중) 사용*
```java
/**
 * PathVariable 사용
 * 변수명이 같으면 생략 가능
 * @PathVariable("userId") String userId -> @PathVariable userId
 */
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
     log.info("mappingPath userId={}", data);
     return "ok";
}

/**
 * PathVariable 사용 다중
 */
@GetMapping("/mapping/users/{userId}/orders/{orderId}")
public String mappingPath(@PathVariable String userId, @PathVariable Long orderId) {
        log.info("mappingPath userId={}, orderId={}", userId, orderId);
        return "ok";
}
```

*특정 헤더 조건 매핑*
```java
/**
 * 특정 헤더로 추가 매핑
 * headers="mode",
 * headers="!mode"
 * headers="mode=debug"
 * headers="mode!=debug" (! = )
 */
@GetMapping(value = "/mapping-header", headers = "mode=debug")
public String mappingHeader() {
     log.info("mappingHeader");
     return "ok";
}
```

*미디어 타입 조건 매핑*

*Content-Type, consume*
```java
/**
 * Content-Type 헤더 기반 추가 매핑 Media Type
 * consumes="application/json"
 * consumes="!application/json"
 * consumes="application/*"
 * consumes="*\/*"
 * MediaType.APPLICATION_JSON_VALUE
 * 
 * consumes = "text/plain"
 * consumes = {"text/plain", "application/*"}
 * consumes = MediaType.TEXT_PLAIN_VALUE
 */
@PostMapping(value = "/mapping-consume", consumes = "application/json")
public String mappingConsumes() {
     log.info("mappingConsumes");
     return "ok";
}
```

*Content-Type, produce*
```java
/**
 * Accept 헤더 기반 Media Type
 * produces = "text/html"
 * produces = "!text/html"
 * produces = "text/*"
 * produces = "*\/*"
 * 
 * produces = "text/plain"
 * produces = {"text/plain", "application/*"}
 * produces = MediaType.TEXT_PLAIN_VALUE
 * produces = "text/plain;charset=UTF-8"
 */
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProduces() {
     log.info("mappingProduces");
     return "ok";
}
```

## HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form

### 클라이언트에서 서버로 요청 데이터를 전달할 때는 주로 다음 3가지 방법을 사용한다.

* **GET - 쿼리 파라미터**
  * /url?username=hello&age=20
  * 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
  * e.g) 검색, 필터, 페이징 등에서 많이 사용하는 방식
  

* **POST**
  * content-type: application/x-www-form-urlencoded
  * 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello&age=20
  * e.g) 회원가입, 상품 주문, HTML Form 사용
  

* **HTTP message Body에 데이터를 직접 담아서 요청**
  * HTTP API 에서 주로 사용, JSON, XML, TEXT
  * 데이터 형식은 주로 JSON 사용
  * POST, PUT, PATCH
<br><br>

## HTTP 요청 파라미터 - @RequestParam

`@RequestParam`: 파라미터 이름으로 바인딩
`@ResponseBody`: View조회를 무시하고, HTTP message body에 직접 해당 내용 입력<br><br>

* @RequestParam("username") String username -> request.getParameter("username")
```java
/**
 * @RequestParam 사용
 * - 파라미터 이름으로 바인딩
 * @ResponseBody 추가
 * - View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
 */
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
   @RequestParam("username") String memberName,
   @RequestParam("age") int memberAge) {
   log.info("username={}, age={}", memberName, memberAge);
   return "ok";
}
```
<br><br>
* HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능
  * @RequestParam("username") String username -> @RequestParam String username
```java
/**
 * @RequestParam 사용
 * HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능
 */
@ResponseBody
@RequestMapping("/request-param-v3")
public String requestParamV3(
   @RequestParam String username,
   @RequestParam int age) {
   log.info("username={}, age={}", username, age);
   return "ok";
}
```
<br><br>

* `String` , `int` , `Integer` 등의 `단순 타입`이면 `@RequestParam` 도 생략 가능
  * 권장하지는 않음(생략이 너무 많으면 나중에 유지보수에 애로사항이 생길 가능성 있음.)
  * `@RequestParam` 애노테이션을 생략하면 스프링 MVC는 내부에서 `required=false 를 적용`한다.
```java
/**
 * @RequestParam 사용
 * String, int 등의 단순 타입이면 @RequestParam 도 생략 가능
 */
@ResponseBody
@RequestMapping("/request-param-v4")
public String requestParamV4(String username, int age) {
   log.info("username={}, age={}", username, age);
   return "ok";
}
```
<br><br>

**파라미터 필수 여부 - requestParamRequired**
* `requestParam.required`
  * 파라미터 필수 여부
  * `기본값`이 파라미터 필수(`true`)이다.
* `/request-param`요청
  * `username`이 없으므로 `400예외` 발생

**주의! - 파라미터 이름만 사용**
* `/request-param?username=`
* 파라미터 이름만 있고 값이 없는 경우 빈문자로 통과

**주의! - 기본형(primitive)에 null 입력**
* `/request-param` 요청
* @RequestParam(required = false) int age
* *null 을 int 에 입력하는 것은 불가능*(`500 예외` 발생)
* 따라서 null 을 받을 수 있는 Integer 로 변경하거나, 또는 다음에 나오는 defaultValue 사용

```java
/**
 * @RequestParam.required
 * /request-param-required -> username이 없으므로 예외
 *
 * 주의!
 * /request-param-required?username= -> 빈문자로 통과
 *
 * 주의!
 * /request-param-required
 * int age -> null을 int에 입력하는 것은 불가능, 따라서 Integer 변경해야 함(또는 다음에 나오는
defaultValue 사용)
 */
@ResponseBody
@RequestMapping("/request-param-required")
public String requestParamRequired(
   @RequestParam(required = true) String username,
   @RequestParam(required = false) Integer age) {
   log.info("username={}, age={}", username, age);
   return "ok";
}
```
<br><br>


**기본 값 적용 - requestParamDefault** 
* 파라미터에 값이 없는 경우 `defaultValue` 를 사용하면 기본 값을 적용할 수 있다.
* 이미 기본 값이 있기 때문에 required 는 의미가 없다.<br>
* *defaultValue 는 빈 문자의 경우에도 설정한 기본 값이 적용*된다. `/request-param-default?username=`

```java
/**
 * @RequestParam
 * - defaultValue 사용
 *
 * 참고: defaultValue는 빈 문자의 경우에도 적용
 * /request-param-default?username=
 */
@ResponseBody
@RequestMapping("/request-param-default")
public String requestParamDefault(
   @RequestParam(required = true, defaultValue = "guest") String username,
   @RequestParam(required = false, defaultValue = "-1") int age) {
   log.info("username={}, age={}", username, age);
   return "ok";
}
```
<br><br>

**파라미터를 Map으로 조회하기 - requestParamMap**

* 파라미터를 Map, MultiValueMap으로 조회할 수 있다.
* `@RequestParam Map` `Map(key=value)`
* `@RequestParam MultiValueMap` `MultiValueMap(key=[value1, value2, ...]` 
  * ex) (key=userIds, value=[id1, id2])
* 파라미터의 값이 1개가 확실하다면 Map 을 사용해도 되지만, 그렇지 않다면 MultiValueMap 을 사용하자.
```java
/**
 * @RequestParam Map, MultiValueMap
 * Map(key=value)
 * MultiValueMap(key=[value1, value2, ...]) ex) (key=userIds, value=[id1, id2])
 */
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
    log.info("username={}, age={}", paramMap.get("username"),
    paramMap.get("age"));
    return "ok";
}
```
<br><br>

## HTTP 요청 파라미터 - @ModelAttribute
개발을 하면 **요청파라미터를 받아서** 필요한 `객체`를 만들고 그 `객체`에 값을 넣어주어야 한다.
스프링은 이러한 과정을 자동화 해주는 `@ModelAttribute`를 제공한다.

#### *스프링 MVC `@ModelAttribute` 실행*
1. `HelloData`객체 생성
2. 요청 파라미터의 이름으로 `HelloData`객체 프로퍼티를 찾는다.
3. 해당 프로퍼티의 setter를 호출하여 파라미터의 값을 바인딩 한다.
   * e.g) `username`이면 `setusername()`을 찾아서 호출하여 값을 입력
```java
@Data
public class HelloData {
 private String username;
 private int age;
}
```
```java
/**
 * @ModelAttribute 사용
 * 참고: model.addAttribute(helloData) 코드도 함께 자동 적용됨, 뒤에 model을 설명할 때
자세히 설명
 */
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
      log.info("username={}, age={}", helloData.getUsername(),
      helloData.getAge());
      return "ok";
}
```
<br><br>

**@ModelAttribute 생략**

`@ModerAttribute`는 생략할 수 있다.<br>
`@RequestParam`도 생략할 수 있으니 혼란이 발생 할 수 있다.. 😥

스프링은 다음과 같은 규칙을 적용함.
* `String`, `int`, `Integer`같은 단순 타입 = `@RequestParam`
* 나머지 = `ModelAttribute`(argument resolver 로 지정해둔 타입 외)
```java
/**
 * @ModelAttribute 생략 가능
 * String, int 같은 단순 타입 = @RequestParam
 * argument resolver 로 지정해둔 타입 외 = @ModelAttribute
 */
@ResponseBody
@RequestMapping("/model-attribute-v2")
public String modelAttributeV2(HelloData helloData) {
    log.info("username={}, age={}", helloData.getUsername(),
    helloData.getAge());
    return "ok";
}
```
<br><br>

## HTTP 요청 메시지 - 단순 텍스트
요청 파라미터와 다르게, HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우
`@RequestParam`, `@ModelAttribute`를 사용할 수 없다.<br>
(물론, HTML Form 형식으로 전달되는 경우는 요청 파라미터로 인정 된다.)

**HTTP message body**에 데이터를 직접 담아서 요청
* HTTP API에서 주로 사용, JSON, XML, TEXT
* 데이터 형식은 주로 JSON 사용
* POST, PUT, PATCH
<br><br>

**텍스트 메시지를 HTTP 메시지 바디에 담아서 전송하고 읽기**
* HTTP 메시지 바디의 데이터를 `InputStream`을 사용해서 직접 읽을 수 있다.
```java
@PostMapping("/request-body-string-v1")
public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
   ServletInputStream inputStream = request.getInputStream();
   String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
   
   log.info("messageBody={}", messageBody);
   
   response.getWriter().write("ok");
}
```
<br><br>

**Input, Output 스트림, Reader**

스프링 MVC는 다음 파라미터를 지원한다.
* `InputStream(Reader)`: HTTP 요청 메시지 바디의 내용을 직접 조회
* `OutputStream(Writer)`: HTTP 응답 메시지의 바디에 직접 결과 출력

```java
/**
 * InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
 * OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력
 */
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
    String messageBody = StreamUtils.copyToString(inputStream,
    StandardCharsets.UTF_8);
    log.info("messageBody={}", messageBody);
    responseWriter.write("ok");
}
```
<br><br>

**HttpEntity**

스프링 MVC는 다음 파라미터를 지원한다.
* `HttpEntity`: HTTP header, body 정보를 편리하게 조회
  * `메시지 바디` 정보를 직접 조회
  * 요청 파라미터를 조회하는 기능과 관계 없음 `@RequestParam` X, `@ModelAttribute` X
* `HttpEntity는 응답에도 사용` 가능
  * `메시지 바디` 정보 직접 반환
  * `헤더` 정보 포함 가능
  * `view` 조회X

`HttpEntity`를 상속받은 다음 객체들도 같은 기능을 제공한다.

* `RequestEntity`
  * HttpMethod, url 정보가 추가, 요청에서 사용
* `ResponseEntity`
  * HTTP 상태 코드 설정 가능, 응답에서 사용
```java
return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED)
```

```java
/**
 * HttpEntity: HTTP header, body 정보를 편리하게 조회
 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 *
 * 응답에서도 HttpEntity 사용 가능
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 */
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
   String messageBody = httpEntity.getBody();
   log.info("messageBody={}", messageBody);
   return new HttpEntity<>("ok");
}
```
<br><br>

**@RequestBody, @ResponseBody**

**@RequestBody**

`@RequestBody`를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다.<br>
**헤더 정보가 필요**하다면 `HttpEntity`를 사용하거나 `@RequestHeader`를 사용하면 된다.<br>
이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 `@RequestParam`, `@ModelAttribute`와는 전혀 관계가 없다.<br>

**@ResponseBody**

`@ResponseBody`를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다.<br>
물론 이 경우에도 `view`를 사용하지 않는다.

* 바디 정보를 조회?
  * `@RequestBody`
* 헤더 정보를 조회?
  * `@RequestHeader` 또는 `HttpEntity`
* 요청 파라미터 조회?
  * `RequestParam` 또는 `@ModelAttribute`
* 바디 정보로 반환?
  * `@ResponseBody`

  
```java
/**
 * @RequestBody
 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 *
 * @ResponseBody
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 */
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) {
   log.info("messageBody={}", messageBody);
   return "ok";
}
```
<br><br>

## HTTP 요청 메시지 - JSON
JSON 데이터 형식을 조회해보자.

`HttpServletRequest`를 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어와서, 문자로 변환한다.<br>
문자로 된 JSON 데이터를 Jackson 라이브러리인 `objectMapper`를 사용해서 자바 객체로 변환한다.

```java
/**
 * {"username":"hello", "age":20}
 * content-type: application/json
 */
@Slf4j
@Controller
public class RequestBodyJsonController {
  /**
   * JSON 데이터를 자바 객체로 변환
   */
  private ObjectMapper objectMapper = new ObjectMapper();
 
 @PostMapping("/request-body-json-v1")
 public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
       ServletInputStream inputStream = request.getInputStream();
       String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
       log.info("messageBody={}", messageBody);
       
       HelloData data = objectMapper.readValue(messageBody, HelloData.class);
       log.info("username={}, age={}", data.getUsername(), data.getAge());
       
       response.getWriter().write("ok");
    }
}
```
<br><br>

**@RequestBody 문자 변환**

`@RequestBody`를 사용해서 HTTP 메시지에서 데이터를 꺼내고 `messageBody`에 저장한다.<br>
문자로 된 JSON 데이터인 `messageBody`를 `objectMapper`를 통해서 자바 객체로 변환한다.<br>

```java
/**
 * @RequestBody
 * HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 *
 * @ResponseBody
 * - 모든 메서드에 @ResponseBody 적용
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 */
@ResponseBody
@PostMapping("/request-body-json-v2")
public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {
   HelloData data = objectMapper.readValue(messageBody, HelloData.class);
   log.info("username={}, age={}", data.getUsername(), data.getAge());
   return "ok";
}
```

*문자로 변환하고 다시 json으로 변환하는 과정이 불편*하다.<br>
`@ModelAttribute`처럼 **한번에 객체로 변환**할 수는 없을까? 🧐<br>

<br><br>

**@RequestBody 객체 변환**

**`@RequestBody` 객체 파라미터**
* `@RequestBody`에 직접 만든 객체를 지정 할 수 있다.
* `HttpEntity`, `@RequestBody`를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 반환해준다.
* HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환해준다.

```java
/**
 * @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
 * HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (content-type: application/json)
 */
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData data) {
   log.info("username={}, age={}", data.getUsername(), data.getAge());
   return "ok";
}
```

**`@RequestBody`는 생략 불가능!!**<br>
스프링은 `@ModelAttribute`, `@RequestParam`과 같은 해당 애노테이션을 생략시 다음과 같은 규칙을 적용
* `String`, `int`, `Integer` 같은 단순 타입 = `@RequestParam`
* 나머지 = `@ModelAttribute` (argument resolver 로 지정해둔 타입 외)

따라서 이 경우 `HelloData`에 `@RequestBody`를 생략하면 `@ModelAttribute`가 적용되어버린다.<br>
* `HelloData data` -> `@ModelAttribute HelloData data`

따라서 **생략하면** HTTP 메시지 바디가 아니라 **요청 파라미터를 처리**하게 된다.<br>

🙉 **주의** 🙉

HTTP 요청시에 `content-type`이 **`application/json`인지 꼭! 확인**해야 한다.<br>
그래야 JSON을 처리할 수 있는 HTTP 메시지 컨버터가 실행된다.<br><br>

**HttpEntity**

`@ResponseBody`
* 응답의 경우에도 `@ResponseBody` 를 사용하면 해당 객체를 HTTP 메시지 바디에 직접 넣어줄 수 있다.
* 물론 이 경우에도 `HttpEntity` 를 사용해도 된다.

`@RequestBody` 요청
* JSON 요청 -> HTTP 메시지 컨버터 -> 객체

`@ResponseBody` 응답
* 객체 -> HTTP 메시지 컨버터 -> JSON 응답

```java
@ResponseBody
@PostMapping("/request-body-json-v4")
public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) {
   HelloData data = httpEntity.getBody();
   log.info("username={}, age={}", data.getUsername(), data.getAge());
   return "ok";
}
```
```java
/**
 * @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
 * HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (contenttype: application/json)
 *
 * @ResponseBody 적용
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter 적용
(Accept: application/json)
 */
@ResponseBody
@PostMapping("/request-body-json-v5")
public HelloData requestBodyJsonV5(@RequestBody HelloData data) {
   log.info("username={}, age={}", data.getUsername(), data.getAge());
   return data;
}
```
<br><br>

## HTTP 응답 - 정적 리소스, 뷰 템플릿
스프링(서버)에서 응답 데이터를 만드는 방법은 크게 3가지 이다.
1. 정적리소스
   * HTML, css, js를 제공할 때 
2. 뷰 템플릿 사용
   * 동적인 HTML을 제공할 때
3. HTTP 메시지 사용
   * HTTP API를 제공하는 경우
   * HTTP 메시지 바디에 JSON과 같은 형식으로 데이터를 실어 보낸다.

### 정적 리소스
`/src/main/resources`는 리소스를 보관하는 곳이고, 또한 클래스 패스의 시작 경로이다.<br>
`/static`, `/pulbic`, `/resources`, `/META-INF/resources` 같은 디렉토리에 정적 리소스를 제공<br>
**정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것이다.**

### 뷰 템플릿
뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다.<br>
일반적으로 HTML을 동적으로 생성하는 용도로 사용한다.<br>
추가로, 뷰 템플릿이 만들 수 있는 것이라면 뭐든지 가능!<br>
뷰 템플릿 경로: `src/main/resources/templates`

**thymeleaf의 뷰템플릿**
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
 <meta charset="UTF-8">
 <title>Title</title>
</head>
<body>
<p th:text="${data}">empty</p>
</body>
</html>
```
`src/main/resources/templates/response/hello.html`<br>
스프링 부트가 자동으로 ThymeleafViewResolver 와 필요한 스프링 빈들을 등록

**뷰 템플릿을 호출하는 컨트롤러**
```java
@Controller
public class ResponseViewController {
     @RequestMapping("/response-view-v1")
     public ModelAndView responseViewV1() {
     ModelAndView mav = new ModelAndView("response/hello").addObject("data", "hello!");
     return mav;
 }
     @RequestMapping("/response-view-v2")
     public String responseViewV2(Model model) {
     model.addAttribute("data", "hello!!");
     return "response/hello";
 }
     @RequestMapping("/response/hello")
     public void responseViewV3(Model model) {
     model.addAttribute("data", "hello!!");
 }
}
```

### String을 반환하는 경우 - View or HTTP 메시지
* `@ResponseBody`가 **없으면** 
  * `response/hello`로 `뷰 리졸버가 실행`되어서 `뷰`를 찾고, `렌더링` 한다.<br>
* `@ResponseBody`가 **있으면** 
  * `뷰 리졸버를 실행하지 않고`, `HTTP 메시지 바디`에 직접 `response/hello` 라는 `문자가 입력`된다.

### Void를 반환하는 경우
* `@Controller` 를 사용하고, `HttpServletResponse`, `OutputStream(Writer)` 같은 **HTTP 메시지 
바디를 처리하는 파라미터가 없으면** `요청 URL을 참고해서 논리 뷰 이름으로 사용`
  * **요청 URL**: `/response/hello`
  * **실행**: `templates/response/hello.html`
* 참고로 이 방식은 명시성이 너무 떨어지고 이렇게 딱 맞는 경우도 많이 없어서, `권장하지 않는다.`

### HTTP 메시지
`@ResponseBody`, `HttpEntity`를 사용하면, 뷰 템플릿을 사용하는 것이 아니라, `HTTP 메시지 바디에
직접 응답 데이터를 출력`할 수 있다.

## HTTP 응답 - HTTP API, 메시지 바디에 직접 입력
HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, 
HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다.

```java
@Slf4j
@Controller
//@RestController
public class ResponseBodyController {
  /**
   * 서블릿을 직접 다룰 때 처럼
   * HttpServletResponse 객체를 통해서 HTTP 메시지 바디에 직접 ok 응답 메시지를 전달한다.
   */
  @GetMapping("/response-body-string-v1")
   public void responseBodyV1(HttpServletResponse response) throws IOException {
      response.getWriter().write("ok");
   }
   
   /**
    * ResponseEntity 엔티티는 HttpEntity 를 상속 받았는데, 
    * HttpEntity는 HTTP 메시지의 헤더, 바디 정보를 가지고 있다. 
    * ResponseEntity는 여기에 더해서 HTTP 응답 코드를 설정할 수 있다.
    * 
    * HttpStatus.CREATED 로 변경하면 201 응답이 나가는 것을 확인할 수 있다.
    */
   @GetMapping("/response-body-string-v2")
   public ResponseEntity<String> responseBodyV2() {
      return new ResponseEntity<>("ok", HttpStatus.OK);
   }

  /**
   * @ResponseBody를 사용하면 view를 사용하지 않고, 
   * HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접 입력할 수 있다.
   * ResponseEntity도 동일한 방식으로 동작한다.
   */
   @ResponseBody
   @GetMapping("/response-body-string-v3")
   public String responseBodyV3() {
      return "ok";
   }

  /**
   * ResponseEntity 를 반환한다. 
   * HTTP 메시지 컨버터를 통해서 JSON 형식으로 변환되어서 반환된다.
   */
  @GetMapping("/response-body-json-v1")
   public ResponseEntity<HelloData> responseBodyJsonV1() {
     HelloData helloData = new HelloData();
     helloData.setUsername("userA");
     helloData.setAge(20);
     
     return new ResponseEntity<>(helloData, HttpStatus.OK);
   }

  /**
   * ResponseEntity 는 HTTP 응답 코드를 설정할 수 있는데, 
   * @ResponseBody 를 사용하면 이런 것을 설정하기 까다롭다.
   * 
   * @ResponseStatus(HttpStatus.OK) 애노테이션을 사용하면 응답 코드도 설정할 수 있다.
   * 
   * 물론 애노테이션이기 때문에 응답 코드를 동적으로 변경할 수는 없다. 
   * 프로그램 조건에 따라서 동적으로 변경하려면 ResponseEntity 를 사용하면 된다.
   */
   @ResponseStatus(HttpStatus.OK)
   @ResponseBody
   @GetMapping("/response-body-json-v2")
   public HelloData responseBodyJsonV2() {
     HelloData helloData = new HelloData();
     helloData.setUsername("userA");
     helloData.setAge(20);
     
     return helloData;
   }
}
```
### @RestController
`@Controller` 대신에 `@RestController` 애노테이션을 사용하면, 해당 컨트롤러에 모두 `@ResponseBody`가 적용되는 효과가 있다.<br>
따라서 _뷰 템플릿을 사용하는 것이 아니라_, **HTTP 메시지 바디에 직접 데이터를 입력**한다. <br>
이름 그대로 `Rest API(HTTP API)를 만들 때 사용하는 컨트롤러`이다.<br>
참고로 `@ResponseBody`는 클래스 레벨에 두면 전체 메서드에 적용되는데,
`@RestController` 에노테이션 안에 `@ResponseBody`가 적용되어 있다.

## HTTP 메시지 컨버터
템플릿으로 HTML을 생성해서 응답하는 것이 아니라, 
**HTTP API처럼 JSON 데이터를 `HTTP 메시지 바디`에서 직접 읽거나 쓰는 경우 
`HTTP 메시지 컨버터`를 사용하면 편리**하다.

<img alt="@ResponseBody 사용 원리.png" src="@ResponseBody 사용 원리.png"/>

`@ResponseBody` 사용 원리
* HTTP 바디에 문자 내용 직접 반환
* `viewResolver` 대신에 `HttpMessageConverter` 동작
* 기본 문자 처리
  * `StringHttpMessageConverter`
* 기본 객체 처리
  * `MappingJsackson2HttpMessageConverter`
* byte 처리 등등
  * `HttpMessageConverter`가 기본으로 등록

응답의 경우 클라이언트의 HTTP Accept헤더와 서버의 컨트롤러 반환 타입 정보 둘을 조합해서
`HttpMessageConverter`가 선택된다.<br>

### 스프링 MVC가 HTTP 메시지 컨버터를 적용하는 경우
* HTTP 요청
  * `@RequestBody`, `HttpEntity(RequestEntity)`
* HTTP 응답
  * `@ResponseBody`, `HttpEntity(ResponseEntity)`

### 스프링 부트 기본 메시지 컨버터
스프링 부트는 다양한 메시지 컨버터를 제공하는데, **대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정**한다.<br>
만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.<br>
몇 가지 주요 컨버터를 알아보자.

**컨버터 종류**
* `ByteArrayHttpMessageConverter`: `Byte[]` 데이터를 처리한다.
  * 클래스 타입: `byte[]` , 미디어타입: `*/*` 
  * 요청 예)`@RequestBody byte[] data`
  * 응답 예) `@ResponseBody return byte[]` 쓰기 미디어타입 application/octet-stream


* `StringHttpMEssageConverter`: `String` 문자로 데이터를 처리한다.
  * 클래스 타입: `String` , 미디어타입: `*/*`
  * 요청 예) `@RequestBody String data` 
  * 응답 예) `@ResponseBody return "ok"` 쓰기 미디어타입 `text/plain`


* `MappingJackson2HttpMessageConverter`: application/json
  * 클래스 타입: `객체` 또는 `HashMap` , 미디어타입 `application/json` 관련 
  * 요청 예) `@RequestBody HelloData data` 
  * 응답 예) `@ResponseBody return helloData` 쓰기 미디어타입 `application/json` 관련

### HTTP 요청 데이터 읽기
* HTTP 요청이 오고, 컨트롤러에서 `@RequestBody` , `HttpEntity` 파라미터를 사용한다.
* `메시지 컨버터`가 메시지를 읽을 수 있는지 확인하기 위해 `canRead()`를 호출한다.
  * 대상 `클래스 타입을 지원`하는가.
    * 예) `@RequestBody` 의 대상 클래스 ( `byte[] , String , HelloData` )
  * HTTP 요청의 `Content-Type 미디어 타입을 지원`하는가.
    * 예) `text/plain` , a`pplication/json` , `*/*`
* canRead() 조건을 만족하면 `read()를 호출해서 객체 생성하고, 반환`한다.

### HTTP 응답 데이터 생성
* 컨트롤러에서 `@ResponseBody`, `HttpEntity` 로 값이 반환된다.
* `메시지 컨버터`가 메시지를 쓸 수 있는지 확인하기 위해 `canWrite()` 를 호출한다.
  * 대상 `클래스 타입을 지원`하는가.
    * 예) return의 대상 클래스 (`byte[] , String , HelloData`)
  * HTTP 요청의 `Accept 미디어 타입을 지원`하는가.**(더 정확히는 @RequestMapping 의 produces)**
    * 예) `text/plain`, `application/json`, `*/*`
* canWrite() 조건을 만족하면 `write() 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성`한다.


## 요청 매핑 핸들러 어댑터 구조
HTTP 메시지 컨버터는 스프링 MVC 어디쯤에서 사용되는 것인가?!!!

`@RequestMapping`을 처리하는 핸들러 어댑터인 `RequestMappingHandlerAdapter`(요청 매핑 핸들러 어댑터)에 있다.

### RequestMappingHandlerAdapter 동작방식

<img alt="RequestMappingHandlerAdapter 동작 방식.png" src="RequestMappingHandlerAdapter 동작 방식.png"/>

**ArgumentResolver**
애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할수 있다.<br>
`HttpServletRequest`, `Model`, `@RequestParam`, `@ModelAttribute`, `@RequestBody`, `HttpEntity` 같은 
`HTTP 메시지`를 처리하는 부분까지 매우 큰 유연함을 갖고 있다.<br>
이렇게 처리할 수 있는 이유가 바로 `ArgumentResolver` 덕분이다.!<br>

애노테이션 기반 컨트롤러를 처리하는 `RequestMappingHandlerAdapter`는 바로
`ArgumentResolver`를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다.

**동작방식**
1. `ArgumentResolver`의 `supporsParameter()`를 호출
2. 해당 파라미터 지원 확인
3. 지원하면 `resolveArgument()`호출하여 실제 객체 생성
4. 생성된 객체가 컨트롤러 호출시 반환

**ReturnValueHandler**

`HandlerMethodReturnValueHandler`를 줄여서 `ReturnVlauteHandler`라 부른다.<br>
`ArgumentResolver`와 비슷한데, 이것은 응답 값을 반환하고 처리한다.<br>
컨트롤러에서 `String`으로 `뷰 이름`을 반환해도, 동작하는 이유가 *바로바로바로! `ReturnValueHandler` 덕분*이다.<br>

스프링은 10여개가 넘는 ReturnValueHandler 를 지원한다.<br>
예) `ModelAndView` , `@ResponseBody` , `HttpEntity` , `String`
* https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types
<br><br>

### HTTP 메시지 컨버터

그렇다면,, `HTTP 메시지 컨버터`는 어디에 위치해 있는가?

HTTP 메시지 컨버터를 사용하는 `@RequestBody` 도 컨트롤러가 필요로 하는 파라미터의 값에 사용된다.
`@ResponseBody` 의 경우도 컨트롤러의 반환 값을 이용한다.

<img alt="HTTP 메시지 컨버터.png" src="HTTP 메시지 컨버터.png"/>

다음 2가지 경우에 대해섯 생각해보자.<br>
<br>

**요청**

`@RequestBody` 를 처리하는 `ArgumentResolver` 가 있고, `HttpEntity` 를 처리하는
`ArgumentResolver` 가 있다. 이 `ArgumentResolver` 들이 `HTTP 메시지 컨버터`를 사용해서 **필요한
객체를 생성**하는 것이다.

**웅답**

`@ResponseBody` 와 `HttpEntity` 를 처리하는 `ReturnValueHandler` 가 있다. 그리고
여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.<br><br>

### 스프링 MVC는 
`@RequestBody`, `@ResponseBody` 가 있으면 
`RequestResponseBodyMethodProcessor (ArgumentResolver)`
`HttpEntity` 가 있으면 `HttpEntityMethodProcessor (ArgumentResolver)`를 사용한다.
