---
title: "406 Not Acceptable HttpMediaTypeNotAcceptableException"
---

Spring Boot 기반 프로젝트를 하면서 406 Not Acceptable HttpMediaTypeNotAcceptableException 에러를 만났었습니다

별거 안했는데 왜 406 에러를 Response 했을까요..??


{% raw %}![alt](https://raw.githubusercontent.com/zziri/zziri.github.io/main/assets/images/2023-10-07/img1.png){% endraw %}
평범한 요청인데 406 Error Response 를 하고 있습니다



에러의 내용을 찍어보고자 ExceptionHandler 를 만들어서 ex.printStackTrace() 메서드를 호출했습니다


```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(value = HttpMediaTypeNotAcceptableException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ResponseDto<ErrorDto> handleHttpMediaTypeNotAcceptableException(HttpMediaTypeNotAcceptableException ex) {
        ex.printStackTrace();
        return ResponseDto.<ErrorDto>builder()
                .data(
                        ErrorDto.builder()
                                .code(HttpStatus.INTERNAL_SERVER_ERROR.value())
                                .message("HttpMediaTypeNotAcceptableException").build())
                .success(false).build();
    }
}
```

```shell
2021-08-21 19:36:38.279  INFO 10384 --- [           main] c.zziri.community.CommunityApplication   : Started CommunityApplication in 10.616 seconds (JVM running for 11.627)
2021-08-21 19:36:39.452  INFO 10384 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2021-08-21 19:36:39.453  INFO 10384 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2021-08-21 19:36:39.455  INFO 10384 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 2 ms
org.springframework.web.HttpMediaTypeNotAcceptableException: Could not find acceptable representation
	at org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodProcessor.writeWithMessageConverters(AbstractMessageConverterMethodProcessor.java:315)
	at org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor.handleReturnValue(RequestResponseBodyMethodProcessor.java:183)
	at org.springframework.web.method.support.HandlerMethodReturnValueHandlerComposite.handleReturnValue(HandlerMethodReturnValueHandlerComposite.java:78)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:124)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:895)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
...
```


뭔가 느낌적으로(?) message를 convert 하는 과정에서 문제가 발생한 것 같습니다



제가 만들어둔 ResponseDto를 살펴보니 평소 Getter, Setter 를 그냥 붙였었는데 이때는 붙이지 않았더군요

Getter 메서드가 없으면 속성들에 접근할 수 없어서 JSON 변환할 때 문제가 생기지 않았을까 하는 의심을 하고 @Getter 어노테이션을 마킹해봤습니다

```java
@Builder
@Getter
public class ResponseDto <T> {
    private boolean success;
    private T data;
}
```


{% raw %}![alt](https://raw.githubusercontent.com/zziri/zziri.github.io/main/assets/images/2023-10-07/img2.png){% endraw %}
이제 정상적으로 Response 되는군요



ResponseDto를 return 해서 클라이언트에 전달하려면 JSON으로 변환해야하는데, Getter 메서드가 없으면 private 인 프로퍼티에 접근하는 것이 당연히 불가능하겠죠...

그래서 이 문제는 @JsonProperty 어노테이션을 활용해도 해결이 가능합니다!

만약 이런 추가 작업 없이 JSON 변환을 하고 싶다면 gson 을 사용하면 됩니다

