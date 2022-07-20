# Spring MVC
MVC 1편(2)(김영한)

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


* **GET - 쿼리 파라미터**
  * /url?username=hello&age=20
  * 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
  * e.g) 검색, 필터, 페이징 등에서 많이 사용하는 방식
* **POST**
  * content-type: application/x-www-form-urlencoded
  * 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello&age=20
  * e.g) 회원가입, 상품 주문, HTML Form 사용
* **HTTP message body에 데이터를 직접 담아서 요청**
  * HTTP API 에서 주로 사용, JSON, XML, TEXT
  * 데이터 형식은 주로 JSON 사용
  * POST, PUT, PATCH

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
