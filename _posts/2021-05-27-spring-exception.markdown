---
layout: post
title:  "[Spring Framework] 예외 처리"
date:   2021-05-27 14:30:00 +0900
categories: spring boot
tags: [spring framework, spring boot, exception, 예외처리] 
---

Spring Boot의 오류 처리에 대한 끄적임

## Spring Boot의 기본 에러 처리 전략 
```
    By default, Spring Boot provides an /error mapping that handles all errors in a sensible way, 
    and it is registered as a “global” error page in the servlet container.
```
 - BasicErrorController 컨트롤러를 제공 하며 /error 매핑을 통해 기본적인 에러를 처리 한다.
 
  ```java
    # spring-boot-autoconfigure.jar 파일의 org.springframework.boot.autoconfigure.web.servlet.error
    # BasicErrorController 가 자동으로 등록 되는 것을 확인 할 수 있다. 
    # server.error.path 를 설정 하지 않았다면 /error 가 기본 오류 처리 PATH주소로 지정된다.
    
    @Controller
    @RequestMapping("${server.error.path:${error.path:/error}}")
    public class BasicErrorController extends AbstractErrorController {
        .......
        @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
        public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
            .......
        }        
        @RequestMapping
        public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
            .......
        }
    }
  ```
   
 - errorHtml() 메서드의 경우 produces 조건이 "text/html" 인 경우 ModelAndView ( /error 기본 경로)를 활용 하여 error html 출력 한다.
 
    ![기본 html 에러 처리](/assets/img/spring-exception/html_default_error.png)
    
    - error html의 경우 resources/public/error 폴더 아래 특정 naming 규칙에 따라 에러를 분기 처리 할 수 있다.
        - 4xx 에러 resources/public/error/4xx.html
        - 5xx 에러 resources/public/error/5xx.html
    
 - 그외의 요청에 경우 error() 메서드를 이용하여 구조화된 json 객체를 출력 한다.
 
    ![그외 에러 처리](/assets/img/spring-exception/other_default_error.png)
    
    - Json 응답 결과는 ErrorAttributes의 구현체인 DefaultErrorAttributes를 사용 하여 출력 한다.
    
  ```java
 public class DefaultErrorAttributes implements ErrorAttributes, HandlerExceptionResolver, Ordered {
    ........
    ........
 	private Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
 		Map<String, Object> errorAttributes = new LinkedHashMap<>();
 		errorAttributes.put("timestamp", new Date());
 		addStatus(errorAttributes, webRequest);
 		addErrorDetails(errorAttributes, webRequest, includeStackTrace);
 		addPath(errorAttributes, webRequest);
 		return errorAttributes;
 	}
 }
 ```

 
## 확장 포인트 - ErrorAttributes
 
 - DefaultErrorAttributes 를 상속 받아 bean 으로 등록할 경우 spring boot는 BasicErrorController는 해당 ErrorAttributes 를 사용 한다.
 
  ```java
  @Component
  public class CustomErrorAttributes extends DefaultErrorAttributes {
      @Override
      public Map<String, Object> getErrorAttributes(WebRequest webRequest, ErrorAttributeOptions options) {
          Map<String,Object> result =  super.getErrorAttributes(webRequest, options);
          result.put("greeting", "Hello");
          return result;
      }
  }
 ```

## 확장 포인트 - BasicErrorController
 - ErrorAttributes 와 마찬가지로 BasicErrorController를 상속 받아 구현 한다.

  ```java
    @Controller
    @RequestMapping("${server.error.path:${error.path:/error}}")
    public class CustomExceptionController extends BasicErrorController {
    
        public CustomExceptionController(ErrorAttributes errorAttributes, ServerProperties serverProperties, List<ErrorViewResolver> errorViewResolvers) {
            super(errorAttributes, serverProperties.getError(), errorViewResolvers);
        }
    
        @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
        public ModelAndView errorHtml(HttpServletRequest request,
                                      HttpServletResponse response) {
            return super.errorHtml(request, response);
        }
    
        @Override
        public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
            return super.error(request);
        }
    
    }
 ```

## CustomException 을 구현해 보자 

 - status가 401인 Custom Unauthorized 예외를 만들고 싶다면...?
 
### CustomException 구성

 - 보통의 웹 어플리케이션의 경우 클라이언트에서 에러를 구분 하기 위해 커스텀한 code와 message를 예외에 따라 내려 준다.
 
 ```java
     public class CustomException extends RuntimeException{
         protected String message;
         protected int code;
         protected CustomException(String message, int code){
             super(message);
             this.message = message;
             this.code = code;
         }
         public String getMessage() {
             return this.message;
         }
     
         public int getCode(){
             return this.code;
         }
     }
 ```
 
 - 상황에 맞게 내려줄 Exception을 CustomException을 상속 받아 구현 한다.
 - @ResponseStatus 값을 원하는 status 를 지정 해야 한다.
 
  ```java
    // status 401에 대한 예외 
    @ResponseStatus(value = HttpStatus.UNAUTHORIZED)
    public class CustomUnauthorizedException extends CustomException{
        public CustomUnauthorizedException(String message, int code){
            super(message , code);
        }
    }   
  ```
  ```java
    // status 404에 대한 예외 
    @ResponseStatus(value = HttpStatus.NOT_FOUND)
    public class CustomNotFoundException extends CustomException{
        public CustomNotFoundException(String message, int code){
            super(message , code);
        }
    }
  ```
### CustomException 발생 

 - 비즈니스 로직에서 예외가 발생 할 경우 바로 예외를 던지도록 한다.
 
  ```java
    @Override
    public UserDetails authenticate(String id, String password) throws CustomException {
        AccountUserDetails userDetails = (AccountUserDetails)userDetailsService.loadUserByUsername(id);
        if(!isMatchPassword(password,userDetails.getPassword())){
            throw new CustomUnauthorizedException("패스워드가 틀렸습니다.",4010001);
        }
        return userDetails;
    }
  ```

### CustomException 처리
 - DefaultErrorAttributes를 상속받아 커스텀 한 에러인 경우 별도의 로직 으로 응답 결과를 구성 한다.
   
 ```java
 @Component
 public class CustomErrorAttributes extends DefaultErrorAttributes { 
    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, ErrorAttributeOptions options) { 
        Map<String,Object> result =  super.getErrorAttributes(webRequest, options);
        Throwable throwable = getError(webRequest);
        if(throwable instanceof CustomException){
            CustomException customException = (CustomException)throwable;
            result.put("custom_message : ", customException.getMessage());
            result.put("custom_code : ", customException.getCode());
        }
        return result;
     }
 }
 ```

 - 결과 
 
 ![custom exception](/assets/img/spring-exception/unauthorized_exception.png)
 
 

## spring-mvc Exception 기반 으로 오류 처리 

 - 위 ErrorController의 경우 해당 Servlet에서 오류를 처리 하지 않아 Servlet container까지 
 오류가 전파되 었을때 Servlet Container가 ErrorController를 호출 한다. 
 
 - Spring 에서는 Handler(Controller의 @RequestMapping 이 걸린 메서드)에서 처리하다 예외가 발생 하였을때,
 이를 Servlet container 까지 전파하지 않고, 직접 예외를 처리 할 수 있도록 해준다.

### @ExceptionHandler 

 - Spring 에서 발생한 에러를 처리 하기 위해 @ExceptionHandler를 제공 한다.
 
 ```java
 @Controller
 @RequestMapping(value = "/hello")
 public class HelloController {
     @ResponseStatus(HttpStatus.UNAUTHORIZED)
     @ExceptionHandler(CustomUnauthorizedException.class)
     public Map<String, Object> exceptionHandle(CustomUnauthorizedException e){
         Map<String, Object> errorAttributes = new HashMap<>();
 
         errorAttributes.put("custom code",e.getCode());
         errorAttributes.put("custom message",e.getMessage() );
 
         return errorAttributes;
     }
     .......
 } 
 ```
 - ErrorAttributes를 확장 해서 사용할 경우 @ExceptionHandler 처리 후 ErrorAttributes가 다시 처리 된다.
 
### @ControllerAdvice
 - @Controller 로 등록되는 Bean 들을 선택적 혹은 전역 으로 공통으로 예외 처리를 할 수 있도록 @ControllerAdvice 를 제공한다.
 ```java
  @ControllerAdvice
  public class CustomControllerAdvice {
      @ResponseStatus(HttpStatus.UNAUTHORIZED)
      @ExceptionHandler(CustomUnauthorizedException.class)
      public Map<String,Object> handle(CustomUnauthorizedException e){
          Map<String, Object> errorAttributes = new HashMap<>();
  
          errorAttributes.put("custom code",e.getCode());
          errorAttributes.put("custom message",e.getMessage() );
  
          return errorAttributes;
      }
  }
 ```

 - @ExceptionHandler와 @ControllerAdvice가 동시에 적용되어 있다면 @ExceptionHandler가 우선적 으로 적용 되고 @ControllderAdvice는 무시 된다.
 
## Filter 와 Interceptor 

 - Filter와 Interceptor는 실행되는 위치가 다르다. 때문에 Exception이 발생 했을 때 처리하는 방법도 달라진다.
 
 - Interceptor는 DispacherServlet 내부에서 발생하기 때문에 ControllerAdvice를 적용 할 수 있다.
 
 - Filter는 DispatcherServlet 외부에서 발생해서 ErrorController 에서 처리 해야 한다. 
 
  ![custom exception](/assets/img/spring-exception/spring_request_lifecycle.jpg)
  
 - 대부분의 비즈니스 로직은 DispatcherServlet 을 거친 다음 실행 되므로 @ControllerAdvice 를 통해 대부분의 예외는 처리 가능할 것으로 보인다.
 