---
title: 인프런 스프링 부트와 JPA 활용1 김영한 강의 정리 3
categories: [Computer engineering, Backend engineering]
tags: [백엔드, 스프링, 자바, backend, spring, java, JPA]
---

Backend engineering을 공부하는 학생이라면 한번쯤 들어봤을 [김영한 님의 실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard)를 들으며 기억해두면 도움될 만한 내용을 정리할 예정입니다. 강의를 모두 수강할 때까지 본 강의 정리 포스팅은 버전을 업데이트하며 주기적으로 업로드 할 예정입니다. 하지만, 주로 기록 목적의 포스팅이다보니 강의 목차 흐름에 따라 내용이 순차적으로 정리되어 다소 본문 내용의 일관성은 떨어질 수 있음을 미리 참고바랍니다. 

## JPA를 활용한 MemberService 테스트 코드 예시
> JPA를 활용하여 MemberService를 작성한 테스트 코드 예시입니다. 아래 코드에서 사용되는 핵심 개념들을 살펴보겠습니다. 우선 아래 코드는 spring이랑 완전히 integration해서 테스트 하기 위해 @Runwith(SpringRunner.class)와 @SpringBootTest 어노테이션을 붙여줬습니다. 또한 테스트 코드 상에 @Transactional 어노테이션이 있으면 테스트가 끝난 뒤 롤백이 되는데 그래서 나중에 테스트 코드를 돌린 로그를 살펴보면 Insert 쿼리가 나가지 않는 것을 확인해 볼 수 있습니다. 정확히 말하면 EntityManager가 flush를 안해서 DB에 커밋이 안되는 것입니다. 그리고 중복_회원_예외() 메소드를 보면 @Test(expected = IllegalStateException.class) 어노테이션이 붙어있는데 이는 아래 메소드 실행 중 해당 예외가 발생하면 정상적으로 return 된 것으로 간주하여 테스트가 잘 끝난 것으로 처리됩니다.   
   
```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.repository.MemberRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)    //spring이랑 완전히 integration해서 테스트하기 위함
@SpringBootTest //spring이랑 완전히 integration해서 테스트하기 위함
@Transactional  //이게 테스트쪽 코드에 있으면 테스트가 끝난 뒤 롤백이 됌.
//그래서 Insert 쿼리가 안나감. 정확히 말하면 EntityManager가 flush를 안해서 DB에 커밋이 안되는 것이다.
public class MemberServiceTest {

    @Autowired
    MemberService memberService;
    @Autowired
    MemberRepository memberRepository;

    @Test
    public void 회원가입() throws Exception {
        //given
        Member member = new Member();
        member.setName("kim");

        //when
        Long saveId = memberService.join(member);

        //then
        assertEquals(member, memberRepository.findOne(saveId));

    }

    @Test(expected = IllegalStateException.class)
    public void 중복_회원_예외() throws Exception {
        //given
        Member member1 = new Member();
        member1.setName("kim1");

        Member member2 = new Member();
        member2.setName("kim1");
        //when
        memberService.join(member1);
        memberService.join(member2);

//        try {
//            memberService.join(member2);
//        } catch (IllegalStateException e) {
//            return;
//        }
        //위 코드가 너무 번거로우면 위처럼     @Test(expected = IllegalStateException.class)라고 작성해주면 된다.
        //이러면 테스트 코드 중 IllegalStateException 예외가 터지면 정상적으로 return되어 테스트가 잘 된 것으로 처리된다.

        //then
        fail("예외가 발생해야 한다.");
    }


}
```

## 메모리 모드 테스트
> 지금까지 테스트는 스프링 부트와 데이터베이스를 모두 띄운 상태로 테스트를 진행했었는데 이러기에는 너무 번거롭습니다. 따라서 메모리 모드로 테스트를 진행하는 방법을 알아보겠습니다. 우선, 스프링 프로젝트 내 test 디렉토리에 **resources/application.yml** 파일을 만들어줍니다. 이렇게 되면 테스트 쪽에서는 **test/resources/application.yml**의 설정을 참조하여 spring이 실행되게 됩니다. 이때 application.yml에 별다른 설정 코드들이 없어도 스프링 부트는 default값으로 메모리 모드로 테스팅을 진행하게 됩니다. 아니면 h2 데이터베이스를 메모리 모드로 맞춰서 테스트를 진행하려면 **application.yml**파일에서 spring:datasource:url을 **jdbc:h2:mem:test**로 수정해주면 됩니다.

## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard