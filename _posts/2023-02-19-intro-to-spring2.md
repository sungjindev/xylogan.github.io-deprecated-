---
title: 인프런 스프링 입문 김영한 강의 정리 2
categories: [Computer engineering, Backend engineering]
tags: [백엔드, 스프링, 자바, backend, spring, java]
---

Backend engineering을 공부하는 학생이라면 한번쯤 들어봤을 [김영한 님의 인프런 스프링 입문 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard)를 들으며 기억해두면 도움될 만한 내용을 정리할 예정입니다. 강의를 모두 수강할 때까지 본 강의 정리 포스팅은 버전을 업데이트하며 주기적으로 업로드 할 예정입니다. 하지만, 주로 기록 목적의 포스팅이다보니 강의 목차 흐름에 따라 내용이 순차적으로 정리되어 다소 본문 내용의 일관성은 떨어질 수 있음을 미리 참고바랍니다. 

## 일반적인 웹 애플리케이션 계층 구조
> 일반적인 웹 애플리케이션은 다음과 같이 4가지로 구성되어 있습니다. 도메인이 우리가 아는 일반적인 객체들이고 이 객체들을 컨트롤러, 서비스, 리포지토리들이 활용하여 애플리케이션을 만듭니다. 요즘은 DDD(Domain Driven Design)라고 하여 도메인 객체 자체가 비즈니스 로직을 갖고 있는 것이 더 나은 경우에 굳이 서비스에 분리하여 구현하지 않고 해당 도메인 객체가 비즈니스 로직을 갖고 있도록 하는 설계도 많이 쓰여 점차 서비스와 도메인의 경계가 허물어지고 있습니다.
* 컨트롤러: 웹 MVC의 컨트롤러 역할
* 서비스: 핵심 비즈니스 로직 구현(예를 들어 회원은 중복 가입이 안된다는 로직 등)
* 리포지토리: 데이터베이스에 접근, 도메인 객체를 DB에 저장하고 관리(저장소라는 의미)
* 도메인: 비즈니스 도메인 객체(예를 들어 회원, 주문, 쿠폰 등처럼 데이터베이스에 주로 저장되고 관리되는 객체)   
   
![1](/assets/img/intro_to_spring/2/1.png){: w="100%" h="100%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

## 인터페이스 구조
> 강의 예제 코드에서는 리포지토리를 구현할 때 어떤 DB를 쓸지 정해지지 않았다는 가정을 하여 인터페이스를 먼저 구현한 뒤에 간단한 메모리 기반의 데이터 저장소를 사용하도록 하였습니다. MemberRepository 인터페이스 코드를 보면 아래와 같습니다.   

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;

import java.util.List;
import java.util.Optional;

public interface MemberRepository {
    Member save(Member member);
    Optional<Member> findById(Long id);
    Optional<Member> findByName(String name);

    List<Member> findAll();
}
```
여기서 Optional이란 Null이 반환될 수 있는 상황에서 NullPointerException(NPE) 에러가 발생하는 경우가 많은데 Optional이라는 Wrapper 클래스를 사용하여 NPE에러를 잘 활용할 수 있도록 해주는 클래스입니다. 위 인터페이스를 활용한 MemoryMemberRepository 구현체 클래스 코드를 보면 아래와 같습니다.   
    
```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;

import java.util.*;

public class MemoryMemberRepository implements MemberRepository {

    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L;

    @Override
    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public Optional<Member> findByName(String name) {
        return store.values().stream()
                .filter(member -> member.getName().equals(name))
                .findAny();
    }

    @Override
    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }
}
```
    
임의의 숫자인 id와 Member 객체를 Key-Value 쌍으로 저장하기 위해 HashMap을 사용해 주었고 임의의 id를 위해 long 데이터 타입의 변수를 선언해줬습니다. 위 코드에는 나와있진 않지만 Member 클래스 내 id 멤버 변수는 Long Wrapper 클래스로 선언되어 있습니다. 이들의 차이는 long은 데이터 타입 그 자체라 별도의 편리한 내장 메소드와 같은 기능들이 없지만 Long은 더욱 편리하게 사용하기 위해 다양한 내장 메소드를 지원한다는 점입니다. 또한 Long 클래스 내부적으로 value 멤버 변수의 데이터 타입으로 long 형을 사용하고 있어서 위처럼 Long Wrapper 클래스에 long 형을 넘겨주는 것이 가능합니다.   
위 코드에서 추가적으로 알아두면 좋을 것이 findByName() 메소드에서 Java의 람다식을 활용했다는 점입니다. store.values()는 HashMap 구조의 Key-Value에서 value 값들을 불러오는데, 그것이 여기서는 Member들입니다. 그 Member들을 stream한다는 것은 Member 하나하나 loop를 돈다는 것이고 그 과정에서 filter를 씌워 각각 member에 들어있는 name의 값과 파라미터로 넘겨받은 name이 동일한 값인지 확인하여 findAny()를 통해 조건에 맞는 값을 모두 리턴한다는 의미입니다.

## 테스트 코드
> 큰 프로젝트라면 더욱 더 테스트 코드가 중요합니다. 테스트 코드는 test 관련 디렉토리 내에 테스트 하고자 하는 클래스의 경로와 동일한 패키지 구성을 만들고 동일한 클래스명 뒤에 보통 "Test"라는 키워드를 덧붙여 테스트 클래스를 생성합니다. 강의에서는 MemoryMemberRepository를 테스트 하기 위해 MemoryMemberRepositoryTest 클래스를 생성하였습니다. 코드는 아래와 같습니다.
   
```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;

import java.util.List;
import java.util.Optional;

import static org.assertj.core.api.Assertions.*;

class MemoryMemberRepositoryTest {
    MemoryMemberRepository repository = new MemoryMemberRepository();

    @AfterEach  //각 메소드 별로 실행이 끝나고 바로 이어 실행되는 콜백 메소드
    public void afterEach() {
        repository.clearStore();
    }

    @Test   //테스트 하는 메소드 끼리 순서는 보장이 안된다!!! 즉 테스트 메소드는 다 따로 독립적으로 동작하도록 설계해야한다. 근데 여기서 계속 동일한 repository를 활용하고 있어서 에러가 뜸. 즉 테스트 메소드 끝날 때마다 비워줘야함.
    public void save() {
        Member member = new Member();
        member.setName("spring");

        repository.save(member);

        Member result = repository.findById(member.getId()).get();
//        Assertions.assertEquals(member, result); 이건 junit 사용
        assertThat(member).isEqualTo(result);   //이건 assertJ 사용, 이게 더 편하다
    }

    @Test
    public void findByName() {
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        Member member2 = new Member();
        member2.setName("spring2");
        repository.save(member2);
        Member result1 = repository.findByName("spring1").get();//cmd+option+v로 왼쪽 변수 및 자료형 자동 완성, get이 optional 타입을 한번 까서 안에 값을 꺼내주는 것, cmd+shift+enter는 괄호나 세미콜론 자동 완성
//        Assertions.assertThat(result1).isEqualTo(member1); 원래는 이게 맞지만 Assertions을 계속 쓰는 게 귀찮으므로 Assertions쪽에 커서를 옮기고 option+enter를 입력하여 Import를 해주면 그 후 편리하게 사용 가능
         assertThat(result1).isEqualTo(member1);

    }

    @Test
    public void findAll() {
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        Member member2 = new Member();  //여러개 같은 코드 복붙에서 변수명만 다 치환해서 바꾸고 싶을때는 shift + F6 
        member2.setName("spring2");
        repository.save(member2);

        List<Member> result = repository.findAll();

        assertThat(result.size()).isEqualTo(2);
    }
}
```
우선 테스트용 메소드가 여러개가 동일한 repository 객체를 사용하며 테스트를 진행하고 있는데 이렇게 되면 공유 변수가 참조되고, 테스트 메소드 끼리는 순서가 보장되지 않으므로 의도하지 않은 결과가 나오게 됩니다. 이를 방지하기 위해 1개의 테스트 메소드가 끝난 뒤에 공유 변수에 대해 초기화를 시켜주는 작업이 필요한데 AfterEach라는 어노테이션을 활용하여 이를 구현하였습니다. clearStore()라는 메소드는 MemoryMemberRepository 클래스에 멤버 함수로 HashMap을 비워주는 clear() 함수를 통해 구현하였습니다.   
테스트할 때 junit, assertJ 등의 툴을 이용하여 테스트를 편리하게 할 수 있는데 특히 내가 입력한 값과 테스트할 기능을 거친 뒤 값 비교를 할 때 Assertions에서 제공하는 다양한 메소드들을 활용하면 쉽게 테스트할 수 있습니다. 이때 Assertions를 쓰는 것이 귀찮으면 Assertions에 커서를 올리고 "option+enter"키를 입력하여 관련 툴을 Import 시켜주면 Assertions 없이 바로 aasertThat() 메소드를 사용할 수 있습니다. 이와 같이 유용한 IntelliJ 단축키들이 있는데 "cmd+option+v"를 어떤 메소드위에 커서를 올린 후 사용하면 해당 메소드가 리턴하는 반환형에 맞게 변수와 자료형이 왼쪽에 자동 완성되며 "cmd+shift+enter"를 입력하면 뒤따라올 괄호나 세미콜론 등을 자동 완성해줍니다. 이 외에도 변수명만 바뀌는 여러 줄의 동일한 소스코드를 복사 붙여넣기하여 활용할 때 변수명만 치환하기 위해서는 치환할 변수명에 커서를 올리고 "shift+F6"을 입력하여 쉽게 치환할 수 있습니다.   
위 코드에서 알아두어여 할 것으로 우리는 MemoryMemberRepository 클래스의 findByName()과 같은 메소드에서 Null 값에 대한 유연한 처리를 위해 Optional이라는 Wrapper 클래스로 감싸놓았는데 그렇기 때문에 Optional로 감싸진 클래스에서 값을 추출하려면 get()이라는 메소드를 사용하여 값을 추출해야 합니다.

## 서비스 코드
> 비즈니스 로직을 처리하기 위해 서비스 코드를 아래와 같이 작성하였습니다. 회원 가입, 회원 조회와 관련된 메소드를 작성하였습니다.   
   
```java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;

import java.util.List;
import java.util.Optional;

public class MemberService {
    private final MemberRepository memberRepository = new MemoryMemberRepository();

    /**
     * 회원 가입
     */
    public Long join(Member member) {
        //시나리오: 같은 이름이 있는 회원이 있어선 안된다.
        validateDuplicateMember(member);    //ctrl+t를 통해 refactoring 관련된 검색을 할 수 있고 메소드를 검색하여 extract method를 하면 메소드를 뽑아낼 수 있다.

        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName())
                        .ifPresent(member1 -> {
                            throw new IllegalStateException("이미 존재하는 회원입니다.");
                        });
    }


    /**
     * 전체 회원 조회
     */
    public List<Member> findMembers() {     // 서비스 클래스 내에서는 조금 더 비즈니스에 가까운 용어로 이름을 선정한다. 리포지토리는 조금 더 데이터 처리에 가까운 용어로 선정
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```
같은 이름의 회원이 저장되어선 안된다라는 시나리오에 맞춰서 중복성 검사를 해주는 함수를 만들어 주었습니다. memberRepository에서 findByName을 활용하였는데 이 메소드의 리턴값이 Optional이므로 값이 존재하는지 검증하는 ifPresent() 메소드를 활용할 수 있었습니다. 그 후 람다식을 활용하여 존재한다면 중복 회원이 존재한다는 에러를 던져주었습니다. 이렇게 구현 한 다음 해당 로직을 전체 드래그 한 뒤 "ctrl+t"를 입력하면 코드를 리팩토링하기 위한 다양한 기능들을 사용할 수 있는 창이 나타나는데 여기에 method를 검색하면 method를 뽑아낼 수 있는 extract method 기능이 있습니다. 위 코드는 이 기능을 통해 메소드를 뽑아낸 상태입니다.   
서비스 관련 클래스에서는 비즈니스 로직을 처리하는 것이 핵심이기 때문에 메소드나 변수 이름을 정할 때 조금 더 비즈니스에 가까운 용어로 이름을 선정하는 것이 중요합니다. 이와 반대로 리포지토리 클래스는 조금 더 데이터 처리에 가까운 용어를 사용하도록 합니다.  

## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard