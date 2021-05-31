---
layout: post
title:  "[Spring Framework] 정적 리소스 (Static Resources) 설정" 
date:   2021-05-27 14:30:00 +0900
categories: spring boot
tags: [spring framework, spring boot, viewResolver, view] 
---

Spring Framework 웹 프로젝트를 진행하는데 있어서 간혹 FrontEnd를 같이 구현 해야 할 경우가 생긴다.  
FrontEnd 페이지에 필요한 각종 static 파일(이미지, js, css...) 을 서버에 보관 해야 하는데 주로 Apache 서버를 이용 하여 제공 한다.  
Spring Framework 또한 static 파일을 직접 제공 하기 위해 정적 리소스 기능을 제공 하고 있다.

### 기본 Static Resource 경로 
- 기본적으로 아래 리소스들은 /** 요청에 mapping 되어 제공된다.

![등록된 ResourceHttpRequestHandler의 locationValues](/assets/img/spring/static-resource-locations.png)


### WebMvcConfigurer 설정

- mapping 을 변경하고 싶다면 WebMvcConfigurer 를 구현 하는 클래스를 만들어 설정 한다.
    - addResourceHandlers 메서드 override
    - ResourceHandlerRegistry를 통해 리소스 위치와 리소스가 매칭될 url을 등록 한다.
 ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            // static/** 패턴의 요청을 내부 classpath의 /static/ 경로와 매핑 시킨다.
            registry.addResourceHandler("/static/**")
                    .addResourceLocations("classpath:/static/")
                    .setCachePeriod(20);
        }
    }
 ```

### 설정 파일

- application.xxx 설정을 통해 mapping을 변경 할 수 있다.

 ```yaml
    spring:
        mvc:
            static-path-pattern: /static/**
 ```

