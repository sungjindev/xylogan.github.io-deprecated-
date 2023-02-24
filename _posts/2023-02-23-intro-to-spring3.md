---
title: 인프런 스프링 입문 김영한 강의 정리 3
categories: [Computer engineering, Backend engineering]
tags: [백엔드, 스프링, 자바, backend, spring, java]
---

Backend engineering을 공부하는 학생이라면 한번쯤 들어봤을 [김영한 님의 인프런 스프링 입문 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard)를 들으며 기억해두면 도움될 만한 내용을 정리할 예정입니다. 강의를 모두 수강할 때까지 본 강의 정리 포스팅은 버전을 업데이트하며 주기적으로 업로드 할 예정입니다. 하지만, 주로 기록 목적의 포스팅이다보니 강의 목차 흐름에 따라 내용이 순차적으로 정리되어 다소 본문 내용의 일관성은 떨어질 수 있음을 미리 참고바랍니다. 

## 컴포넌트 스캔으로 스프링 빈 등록
> 스프링 어플리케이션 메인 클래스가 있는 패키지 하위에 있는 것들은 스프링이 실행될 때 자동으로 스프링 컨테이너에서 관리될 수 있도록 쭉 스캔한다. 이때 @Component라는 어노테이션이 존재하는 경우 해당 객체를 싱글톤(유일하게 하나만 만들어서 공유)으로 만들어서 스프링 컨테이너에서 스프링 빈으로 관리한다고 표현하게 되는데 컨트롤러 관련 클래스에는 @Controller, 서비스 관련 클래스에는 @Service, 리포지토리는 @Repository 어노테이션을 붙여주면 이 각각의 어노테이션은 모두 @Component 어노테이션을 포함하고 있어서 스프링 빈으로 등록되게 됩니다. 이를 통해 각각 계층에서 공유된 객체들을 쉽게 사용할 수 있습니다. 컨트롤러 클래스에서 생성자를 만들고 @Autowired 어노테이션을 사용하면 서비스 관련 스프링 빈을 활용해 의존성 주입해줄 수 있습니다. 마찬가지로 서비스 클래스에서 생성자를 만들고 @Autowired 어노테이션을 사용하면 리포지토리 관련 스프링 빈을 활용해 의존성 주입해줄 수 있습니다. 이러한 방식을 **컴포넌트 스캔과 자동 의존 관계 설정 방법**이라고 합니다. 활용 예제 코드는 아래와 같습니다.   

![1](/assets/img/intro_to_spring/3/1.png){: w="100%" h="100%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

```java
package hello.hellospring.controller;

import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

@Controller //Controller라는 어노테이션이 있으면 스프링이 실행될 때 스프링 컨테이너에서 관리한다. 이런 걸 스프링 컨테이너에서 스프링 빈이 관리된다라고 표현한다.
public class MemberController {
    private final MemberService memberService;

    @Autowired  //스프링 컨테이너에서 관리하는 MemberService의 객체를 찾아 자동으로 연결시켜주는데 이렇게 사용하려면 MemberService가 스프링 컨테이너에서 관리하도록 해야한다. MemberService는 서비스 이므로 해당 클래스에가서 @Service 어노테이션을 사용하면 된다.
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```

## 자바 코드로 직접 스프링 빈 등록
>  컨트롤러에만 @Controller, @Autowired를 사용하고 그 이외 서비스와 리포지토리에서는 해당 어노테이션들을 사용하지 않고 별도로 메인 어플리케이션 클래스가 존재하는 경로에 **SpringConfig**라는 클래스를 만들어 자바 코드로 직접 스프링 빈 등록을 할 수 있습니다. 코드는 아래와 같습니다.   
   
```java
package hello.hellospring;

import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());   //아래에서 만든 memberRepository() 메소드 호출하여 의존성 주입
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```

## DI(Dependency Injection) 의존성 주입
> 의존성 주입이란 외부에서 객체를 넣어주는 것을 말하는데, 총 3가지의 구현 방법이 있습니다. 넣고싶은 클래스에 필드를 만들어서 필드에 @Autowired를 사용해 스프링 빈을 주입하는 필드 주입 방법과, 넣고싶은 클래스에 생성자를 만들어서 @Autowired를 사용해 스프링 빈을 주입하는 생성자 주입 방법, 그리고 마지막으로 setter를 활용하는 방법이 있습니다. setter를 사용하면 setter가 public으로 노출되어 있어야만해서 중간에 값을 잘못 바꾸는 문제가 생길 수 있고 필드를 활용하게 되면 해당 클래스가 뜰 때만 넣어주게 되서 중간에 바꿀 수 있는 방법이 없습니다. **따라서 생성자를 통한 의존성 주입을 가장 권장합니다.**



## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard