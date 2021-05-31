---
layout: post
title:  "[Spring Framework] DispatcherServlet" 
date:   2021-05-27 14:30:00 +0900
categories: spring boot
tags: [spring framework, spring boot, servlet, DispatcherServlet ] 
---

## DispatcherServlet
 
- FrontController 패턴을 이용 하여 모든 요청들에 대해 다른 Controller 및 Handler 들에게 요청을 위임 하는 역할을 한다.

## 요청과 응답에 대한 Flow 

- DispatcherServlet의 동작 흐름은 아래와 같다.

![DispatcherServlet 동작](/assets/img/spring/spring-dispacher-servlet.png)
 
- 모든 요청은 FrontController를 거치게 된다.
   - 해당 컨트롤러는 Handler Mapping에 미리 정의 (등록) 되어 있는 Controller 객체를 찾아 **요청**을 위임 한다.
    
- 요청을 위임 받은 Controller는 요청에 대한 서비스를 처리 하고 나온 **응답**을 다시 FrontController 에게 위임 한다.
   - 응답엔 서비스 데이터와 더불어 뷰에 대한 정보도 함께 포함 되어 있다.
   
-  FrontController는 Controller가 넘겨준 뷰 정보를 어떤 식으로 해석 할지 결정 하게 된다.
   - 이때 미리 등록된 View Resolver 전략중 하나를 선택해 최종적 으로 뷰 오브젝트와 데이터를 바탕 으로 최종 결과 물을 만들게 된다. 
    
- 최종 결과물은 HttpServletResponse 오브젝트에 담겨져 Client로 전송 하게 된다.  

## DispatcherServlet 의 구성 요소 

### 1. HandlerMapping
 - 사용자의 요청을 분석하여 어떤 Controller를 사용 할지 결정 하는 역할을 한다. 
 - Spring Boot의 경우 아래와 같은 HanderMapping을 기본적으로 등록해 준다.
 
![기본 HandlerMapping](/assets/img/spring/default_handler_mapping.png){:height="200px"}

#### 1.1. RequestMappingHandlerMapping 

   - @Controller 혹은 @RequestMapping 어노테이션을 사용하는 Controller 를 선택 한다.   
     실제로 어떤 메서드를 선택할지는 Adapter를 통해 결정 된다. 대부분 해당 HandlerMapping 방식을 사용한다.
   
   ```java
    @Controller
    @RequestMapping("/hello")
    public class HelloController {
        .....
    }
   ```
   - 어떻게 자동 으로 등록 하는 걸까...?
   
   ```java
    // AbstractHandlerMethodMapping 클래스
    public abstract class AbstractHandlerMethodMapping<T> extends AbstractHandlerMapping implements InitializingBean {
     // ApplicationContext 에 등록되어 있는 bean 중 @Controller 와 @ReeustMapping이 등록된 bean을 찾아 등록한다. 
        protected void processCandidateBean(String beanName) {
            Class<?> beanType = null;
            try {
                beanType = obtainApplicationContext().getType(beanName);
            }
            catch (Throwable ex) {
                // An unresolvable bean type, probably from a lazy bean - let's ignore it.
                if (logger.isTraceEnabled()) {
                    logger.trace("Could not resolve type for bean '" + beanName + "'", ex);
                }
            }
            if (beanType != null && isHandler(beanType)) {
                detectHandlerMethods(beanName);
            }
        }
    }
   ```
   ```java
    // RequestMappingHandlerMapping 클래스 
    public class RequestMappingHandlerMapping extends RequestMappingInfoHandlerMapping
    		implements MatchableHandlerMapping, EmbeddedValueResolverAware {
       // isHandler 메서드 
       @Override
       protected boolean isHandler(Class<?> beanType) {
       	return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
       			AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
       }
    }
   ```
 - org.springframework.web 패키지의 로그 레벨을 TRACE 로 설정 하면  다음과 같이 로그를 통해 HandlerMapping이 등록되는걸 확인 할 수 있다.
  
 ![기본 HandlerAdapter](/assets/img/spring/request_handler_mapping_log.png){:height="200px"}
 

#### 1.2. BeanNameUrlHandlerMapping
    
   - 정의한 클래스명 그대로 bean name을 url로 사용 한다는 전략 이다.
   
   ```java
    @Bean("/hello")
    public HelloController helloController(){
        return new HelloController();
    }
   ``` 
#### 1.3. SimpleUrlHandlerMapping
   
  - url 과 Controller를 map에 담아 매핑할 수 있는 전략 이다.
    
 ```java
     @Bean
     SimpleUrlHandlerMapping urlHandlerMapping() {
         SimpleUrlHandlerMapping simpleUrlHandlerMapping = new SimpleUrlHandlerMapping();
         simpleUrlHandlerMapping.setOrder(Ordered.LOWEST_PRECEDENCE - 2);
     
         Map<String, Object> mapping = new HashMap<>();
         mapping.put("/hello", helloController());
         simpleUrlHandlerMapping.setUrlMap(mapping);
         return simpleUrlHandlerMapping;
     } 
   ``` 
 
### 2. HandlerAdapter
 - HandlerMapping에서 결정된 핸들러 정보를 통해 해당 메서드를 직접 호출 하는 기능을 한다. 
 - 호출 해야 하는 컨트롤러 타입을 지원하는 어댑터를 중간에 껴서 호출 하는 방식을 사용한다.
 
 ![기본 HandlerAdapter](/assets/img/spring/default_handler_adapter.png){:height="200px"}
 
#### 2.1. RequestMappingHandlerAdapter
 - RequestMappingHandlerMapping과 대응되는 클래스이다.
 - 리플렉션을 이용하여 메서드를 호출 하는 방식 이므로 해당 파라미터 타입과 리턴 타입들의 정보를 가지고 있다.
    (HandlerMethodArgumentResolver, HandlerMethodReturnValueHandler)
 
  ```java
    // 해당 클래스는 RequestMappingHandlerAdapter를 통해 hello(Long) 이란 메서드가 호출 된다.
    @Controller
    @RequestMapping("/hello")
    public class HelloController {
        @GetMapping
        public String hello(@PathVariable Long id) {
           //...
        }
    }
  ``` 
    
#### 2.2. SimpleServletHandlerAdapter
 - Servlet을 직접 구현하여 bean으로 등록해서 사용한다.
  
 ```java
    public class HelloController extends HttpServlet {
    
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    
        }
    }
 ```
   
#### 2.3. SimpleControllerHandlerAdapter
  - Controller 인터페이스를 사용한다. 
  
 ```java
    public class HelloController implements Controller {
        @Override
        public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
            ///... 
            return null;
        }
    }
 ```

#### 2.4. HttpRequestHandlerAdapter

 - HttpRequestHandler 인터페이스를 사용 한다.
 
  ```java
    public class AccountController implements HttpRequestHandler {
    
        @Override
        public void handleRequest(HttpServletRequest request, HttpServletResponse response)  {
            ///...
        }
    }
  ```
 

### 3. HandlerExceptionResolvers
 - DispacherServler 실행 중에 일어나는 예외들에 대한 처리를 위한 Resolver를 등록한다. 자세한건 예외처리 에서...

  ![기본 ExceptionResolver](/assets/img/spring/default_exception_resolver.png){:height="150px"}
  
### 4. ViewResolvers
 - Controller 가 리턴한 View 이름을 참고 해서 적절하게 View Object를 찾아주는 로직을 담당한다.
