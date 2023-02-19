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


## 빌드하고 실행하기
> 터미널 환경에서 스프링 부트를 빌드하고 실행하는 방법입니다.
```console
./gradlew build
cd build/libs
java -jar hello-spring-0.0.1-SNAPSHOT.jar
```

## 정적 컨텐츠
> 정적 컨텐츠 혹은 정적 페이지란 런타임 시에 컨텐츠의 내용이 변경되지 않는 말 그대로 현재 소스 코드 그대로 페이지를 보여주는 것을 의미합니다. 스프링 부트에서는 resources/static 폴더 밑에 담긴 html 파일의 경우 정적 컨텐츠로 활용합니다. 해당 경로에 html 파일을 만들고 "localhost:8080/hello-static.html" 식으로 접근하면 정적 페이지가 나타나게 됩니다.

## MVC와 템플릿 엔진
> MVC는 Model, View, Controller의 약자로 웹을 세 계층으로 분리하여 개발하는 것을 말합니다. Controller에 비즈니스 로직과 관련된 내용들이 처리되고 이때 일부 내용들을 모델에 담아 뷰로 넘겨줍니다. 뷰에서는 이렇게 전달받은 데이터들을 바탕으로 동적으로 페이지 출력에 집중하여 사용자에게 띄워줍니다. 이 내부 흐름은 앞서 설명드린 "스프링 부트 + Thymeleaf 템플릿 엔진 동작 원리"와 동일합니다.

## API
> API 방식은 Controller에 비즈니스 로직과 관련된 내용들이 처리된다는 점이 MVC와 동일하지만 그 이후 템플릿 엔진이 아닌, 문자열 데이터 그대로나 객체 데이터가 JSON 형식 따위로 그대로 Http body에 담아 반환하는 것을 말합니다. MVC와 템플릿 엔진 방식과 다르게 viewResolver를 사용하지 않고 HttpMessageConverter를 사용하며 문자열 데이터의 경우에는 StringHttpMessageConverter를 사용하고 객체 데이터의 경우에는 디폴트로 JSON 변환을 하기 위해 MappingJackson2HttpMessageConverter를 사용하여 처리합니다. 추가적으로 클라이언트 단에서 Http Accept 헤더에 원하는 반환형을 지정할 수 있는데 이 정보와 컨트롤러의 반환 타입 정보를 조합하여 적절한 HttpMessageConverter를 선택하여 사용합니다. 구조는 아래와 같습니다.
![3](/assets/img/intro_to_spring/1/3.png){: w="100%" h="100%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

## MVC와 API를 활용한 컨트롤러 예제 코드
>   

```java
@Controller
public class HelloController {
    @GetMapping("hello-mvc")    //MVC와 템플릿 엔진을 활용한 컨트롤러
    public String helloMvc(@RequestParam("name") String name, Model model) {
        model.addAttribute("name", name);
        return "hello-template";
    }

    @GetMapping("hello-string") //API를 활용해 문자열 데이터 반환, 문자열 형식으로 Http body에 담아 출력
    @ResponseBody
    public String helloString(@RequestParam("name") String name) {
        return "hello " + name; //"hello spring!!!"
    }

    @GetMapping("hello-api")    //API를 활용해 객체 데이터 반환, JSON 형식으로 Http body에 담아 출력
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name) {
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }

    static class Hello {    //객체를 반환하기 위해 static class를 생성하고 getter와 setter를 정의
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }

}
```

## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard