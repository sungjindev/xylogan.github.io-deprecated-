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

## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard