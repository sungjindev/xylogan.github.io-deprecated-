---
title: 인프런 스프링 입문 김영한 강의 정리 4
categories: [Computer engineering, Backend engineering]
tags: [백엔드, 스프링, 자바, backend, spring, java]
---

Backend engineering을 공부하는 학생이라면 한번쯤 들어봤을 [김영한 님의 인프런 스프링 입문 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard)를 들으며 기억해두면 도움될 만한 내용을 정리할 예정입니다. 강의를 모두 수강할 때까지 본 강의 정리 포스팅은 버전을 업데이트하며 주기적으로 업로드 할 예정입니다. 하지만, 주로 기록 목적의 포스팅이다보니 강의 목차 흐름에 따라 내용이 순차적으로 정리되어 다소 본문 내용의 일관성은 떨어질 수 있음을 미리 참고바랍니다. 

## H2 데이터베이스 설치 및 연결
> H2 데이터베이스를 설치해준 뒤 압축을 풀고 "/h2/bin"에 터미널로 들어가 "chmod 755 h2.sh"로 권한을 준 뒤 "./h2.sh"를 실행하면 웹 UI가 하나 나타납니다. 이때 WEB URL을 보면 임의의 IP주소가 들어가 있는데 이 부분을 "localhost"로 치환해주면 됩니다. 그 후 JDBC URL을 "jdbc:h2:tcp://localhost/~/test"로 바꿔주면 소켓을 통해 어플리케이션이랑 웹 컨트롤러 등에서 동시에 충돌 없이 DB에 잘 접근할 수 있게 됩니다. 데이터베이스 관련된 파일은 사용자의 홈 디렉터리에 "test.mv.db"에 저장되어 있는데 연결에 문제가 있다면 "rm test.mv.db"로 한번 지워준 뒤 실행시켰던 "h2.sh"도 중지시켰다가 재실행하고 처음에는 JDBC URL에 "jdbc:h2:~/test"를 입력하여 연결해줬다가 그 이후부터는 아까처럼 충돌 없이 소켓으로 연결하기 위해 "jdbc:h2:tcp://localhost/~/test"로 연결해주면 됩니다. 

## 스프링 통합 테스트
> 스프링 통합 테스트란 테스트 코드를 실행할 때 실제로 스프링과 데이터베이스 등도 함께 띄워 테스트를 진행하는 것을 말합니다. 이를 위해 테스트 코드 클래스에 "@SpringBootTest" 어노테이션을 달아주면 됩니다. 이렇게 되면 실제로 DB에 테스트할 때 사용된 데이터들이 반영될 수 있는데 "@Transactional" 어노테이션을 테스트 케이스에 달면 테스트 시작 전에 트랜잭션을 모두 실행하고 테스트가 끝나면 항상 롤백해주기 때문에 테스트를 아무리 하여도 데이터베이스에 반영시키지 않을 수 있어 테스트 코드 간의 충돌을 막아줄 수 있습니다. 예제 코드는 아래와 같습니다.   
   
```java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

@SpringBootTest
@Transactional //이걸 테스트 케이스에 달면 테스트 실행할 때 트랜잭셔널 실행하고 DB에 데이터를 모두 넣은 다음에 롤백을 해줘서 DB에 테스트한 데이터들이 반영되지 않음
class MemberServiceIntegrationTest {

    @Autowired
    MemberService memberService;
    @Autowired
    MemberRepository memberRepository;

    @Test
    void 회원가입() {
        //given
        Member member = new Member();
        member.setName("spring");

        //when
        Long saveId = memberService.join(member);

        //then
        Member findMember = memberService.findOne(saveId).get();
        assertThat(member.getName()).isEqualTo(findMember.getName());
    }

    @Test
    public void 중복_회원_예외() {
        //given
        Member member1 = new Member();
        member1.setName("spring");

        Member member2 = new Member();
        member2.setName("spring");
        //when
        memberService.join(member1);
        IllegalStateException e = assertThrows(IllegalStateException.class, () -> memberService.join(member2));
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");

    }

}
```

## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard