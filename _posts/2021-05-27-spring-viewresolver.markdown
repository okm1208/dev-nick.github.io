---
layout: post
title:  "[Spring Framework] ViewResolver"
date:   2021-05-27 14:30:00 +0900
categories: spring boot
tags: [spring framework, spring boot, viewResolver, view] 
---

Spring Framework View Resolver란 무엇인가..?

- 어떤한 요청이 있을 경우 그에 대한 응답을 내려 주는 것이 서버의 역할 일 것이다. 
하지만 서버의 응답의 경우 다양한 형태로 view를 랜더링 하여 보여줄 것인가를 결정 해야 할 것이다.

- Spring Framework MVC는 View Resolver를 통해서 응답에 대한 View 랜더링 을 결정 한다.

###  ViewResolver 전략   

 - Spring Boot의 경우 Autoconfiguration 기능을 활용해 기본적인 ViewResolver를 제공한다.
 
 ![WebAutoConfiguration](/assets/img/spring/autoconfiguration_viewsolver.png)

 - ContentNegotiatingViewResolver
    - 요청 파일이나 Accept헤더에 기반한 뷰를 처리 하는 ViewResolver의 구현체 
     
 - InternalResourceViewResolver ( defaultViewResolver )
    - 디폴트로 설정되는 ViewResolver로 Jsp ,Html 파일과 같이 Web Application 내부 Resource를 이용하여 뷰를 생성한다. 
    


### TBD...
 
     