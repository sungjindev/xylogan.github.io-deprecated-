---
title: 인프런 스프링 입문 김영한 강의 정리 1
categories: [Computer engineering, Backend engineering]
tags: [백엔드, 스프링, 자바, backend, spring, java]
---

Backend engineering을 공부하는 학생이라면 한번쯤 들어봤을 [김영한 님의 인프런 스프링 입문 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard)를 들으며 기억해두면 도움될 만한 내용을 정리할 예정입니다. 강의를 모두 수강할 때까지 본 강의 정리 포스팅은 버전을 업데이트하며 주기적으로 업로드 할 예정입니다. 하지만, 주로 기록 목적의 포스팅이다보니 강의 목차 흐름에 따라 내용이 순차적으로 정리되어 다소 본문 내용의 일관성은 떨어질 수 있음을 미리 참고바랍니다. 

## Spring 프로젝트 시작
> Spring 프로젝트를 시작하기 전에 앞서, 본 강의에서는 Java 11 버전을 사용하며, 개발을 위해 IntelliJ IDE를 사용합니다. 기본적인 설정을 모두 마친 뒤, 스프링을 통해 웹 서버 개발을 하기 위해 필요한 라이브러리 들을 불러올 설정들을 입력해줘야 하는데 요즘은 [spring initializr](https://start.spring.io/)를 사용해 쉽게 Web UI를 사용하여 사용자가 커스텀한 설정이 들어간 project 뼈대를 다운로드하여 활용할 수 있습니다. 저는 Java 11 버전을 사용하기 때문에 아래와 같이 설정해 줬습니다.
![1](/assets/img/intro_to_spring/1/1.png){: w="100%" h="100%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

## Gradle 라이브러리 살펴보기
> Gradle은 의존 관계가 있는 라이브러리들을 함께 다운로드하며 관리해줍니다. 그 중에서도 Spring Web과 Spring thymeleaf를 추가했을 때 의존성에 따라 함께 다운로드 되는 핵심적인 라이브러리를 살펴보면 아래와 같습니다.  
   
우선 스프링 부트 관련 라이브러리는 아래와 같습니다.  

* spring-boot-starter-web
    * spring-boot-starter-tomcat: 웹 서버를 띄워주는 톰캣
    * spring-webmvc: 스프링 웹 MVC
* spring-boot-starter-thymeleaf: 타임리프 템플릿 엔진(View)
* spring-boot-starter(공통): 스프링 부트 + 스프링 코어 + 로깅
    * spring-boot
        * spring-core
    * spring-boot-starter-logging
        * logback, slf4j   

테스트 관련 라이브러리는 아래와 같습니다.
* spring-boot-starter-test
    * junit: 테스트 프레임워크
    * mockito: 목 라이브러리
    * assertj: 테스트 코드를 좀 더 편하게 작성할 수 있도록 도와주는 라이브러리
    * spring-test: 스프링 통합 테스트 지원

## 스프링 부트가 제공하는 Welcome page
> "/src/main/resources/static/index.html" 경로로 index.html 파일을 만들어주면 해당 index.html이 welcome page(root page)로 동작하게 됩니다. index.html 파일은 아래와 같은 간단한 예제 코드로 만들어 볼 수 있습니다.  

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
Hello
<a href="/hello">hello</a>
</body>
</html>
```

## 스프링 부트 + Thymeleaf 템플릿 엔진 동작
> 스프링 부트와 Thymeleaf 템플릿 엔진이 서로 어떻게 조화롭게 동작하는지 아래 구조를 보며 이해할 수 있습니다. 스프링 부트에 내장되어 있는 톰캣이 로컬 웹 서버를 띄워주고 @GetMapping("hello")을 가진 helloController가 model에 data라는 이름의 attribute에 "hello!!!"라는 값을 담아놓은 뒤 return "hello"을 통해 viewResolver에게 전달합니다. 컨트롤러에서 문자를 리턴하면 viewResolver가 "resources/templates/"의 경로에 리턴받은 값의 html 파일을 찾아 처리하여 웹 페이지로 띄워줍니다.
![2](/assets/img/intro_to_spring/1/2.png){: w="100%" h="100%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}


## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard