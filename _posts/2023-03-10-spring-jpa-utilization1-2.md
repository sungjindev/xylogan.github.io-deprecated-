---
title: 인프런 스프링 부트와 JPA 활용1 김영한 강의 정리 2
categories: [Computer engineering, Backend engineering]
tags: [백엔드, 스프링, 자바, backend, spring, java, JPA]
---

Backend engineering을 공부하는 학생이라면 한번쯤 들어봤을 [김영한 님의 실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard)를 들으며 기억해두면 도움될 만한 내용을 정리할 예정입니다. 강의를 모두 수강할 때까지 본 강의 정리 포스팅은 버전을 업데이트하며 주기적으로 업로드 할 예정입니다. 하지만, 주로 기록 목적의 포스팅이다보니 강의 목차 흐름에 따라 내용이 순차적으로 정리되어 다소 본문 내용의 일관성은 떨어질 수 있음을 미리 참고바랍니다. 

## @ManyToMany
> JPA에서 N:M 다대다 관계를 매핑할 때 쉽게 할 수 있도록 @ManyToMany 어노테이션을 지원합니다. 결론부터 말하자면 이 방식을 사용하는 것은 정말 비추천합니다. 왜냐하면 JPA의 해당 어노테이션을 통해서 JoinTable을 만들게 되면 만들어주는 거 외적으로 해당 테이블에 필요한 추가적인 컬럼을 커스텀 생성할 수 없기 때문입니다. 그래서 직접 다대다 관계를 1:N과 N:1로 풀어주고 JoinTable로 사용할 엔티티를 별도로 직접 만들어주는 것을 추천합니다. 하지만 그래도 @ManyToMany도 사용법을 한번쯤 알아두면 좋으므로 아래 코드로 알아보겠습니다. @ManyToMany 어노테이션과 함께 @JoinTable 어노테이션도 사용해줘야 하는데 joinColumns에는 현재 속해있는 클래스의 JoinColumn 명을 적어줘야하며 inverseJoinColumns에는 상대 클래스의 JoinColumn을 작성해주면 됩니다.   
   
```java
package jpabook.jpashop.domain;

import jpabook.jpashop.domain.item.Item;
import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Getter @Setter
public class Category {

    @Id @GeneratedValue
    @Column(name = "category_id")
    private Long id;

    private String name;

    @ManyToMany
    @JoinTable(name = "category_item",
        joinColumns = @JoinColumn(name = "category_id"),
            inverseJoinColumns = @JoinColumn(name = "item_id"))  //ManyToMany는 N:M 다대다 관계이므로 RDB 특성상 JoinTable을 활용해서
    //1:N 과 N:1 관계로 풀어내야 함. 어찌됐건 JPA에서 ManyToMany는 쓰지 않는 것이 좋다. 직접 다대다를 1:N과 N:1로 풀어서 구현하는 게 좋다.
    //joinColumns에는 현재 클래스, 즉 Category 클래스의 JoinColumn을 작성해주는 것이고 inverseJoinColumns에는 상대의 JoinColumn을 작성해주는 것이다.
    private List<Item> items = new ArrayList<>();
}
```

## JPA 카테고리 구조 만들기
> JPA를 활용해서 대분류 - 중분류 - 소분류 등의 카테고리 구조를 만드는 방법에 대해서 설명드리겠습니다. 간단히 설명드리면 동일한 엔티티 내부에서 셀프로 양방향 연관 관계를 만들어준다고 생각하면 됩니다. 아래 코드를 보시면 부모를 가리킬 필드와 자식을 가리킬 필드를 적절히 만들어 준 다음 셀프로 연관 관계를 이어준 것을 확인해 볼 수 있습니다. 이렇게 생성하시면 카테고리 구조를 만들 수 있습니다.   
   
```java
package jpabook.jpashop.domain;

import jpabook.jpashop.domain.item.Item;
import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Getter @Setter
public class Category {

    @Id @GeneratedValue
    @Column(name = "category_id")
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Category parent;    //카테고리 구조를 구현하는 방법. 부모, 자식 필드를 만들고 아래와 같이
    //parent와 child를 이용해 셀프로 양방향 연관관계를 걸어주면 된다
    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();


}
```

## 엔티티 설계할 때 주의점
1. 엔티티에는 가급적 Setter를 사용하지 말아야 합니다. Setter가 모두 열려있으면 변경 포인트가 너무 많아져서 유지보수하기가 어렵습니다.
2. 모든 연관 관계는 지연 로딩으로 설정해야 합니다. 지연 로딩과 반대되는 개념인 즉시 로딩은 연관된 데이터를 모두 한번에 가져오는 것을 말합니다. 즉시 로딩(EAGER)은 예측이 어렵고 어떤 SQL이 실행될 지 추적하기도 어렵습니다. 특히 JPQL을 실행할 때 N+1 문제가 자주 발생하게 됩니다. 실무에서 모든 연관 관계는 반드시 지연 로딩(LAZY)으로 설정해야 합니다. 만약 연관된 엔티티를 함께 DB에서 조회해야 한다면, fetch join이나 엔티티 그래프 기능을 사용하면 됩니다. @XToOne의 꼴이 되는 어노테이션들, 즉 @OneToOne 혹은 @ManyToOne를 사용하면 default로 FetchType이 EAGER로 설정되어 있는데 예를 들어 Order가 Member와 @ManyToOne으로 연결되어 있다고 하면 Order를 조회하는 쿼리를 한번 날리면 Member 테이블도 함께 조인해서 한꺼번에 가져오는 기능이 Eager입니다. 이때는 문제가 없지만 만약에 JPQL 같은 것으로 쿼리를 날리게 되면 Order 테이블을 조회하는 쿼리를 1번 날리고 난뒤 예를들어 Order 테이블에 데이터 row가 100개라면 각각 데이터마다 EAGER 타입으로 관계가 맺어져있는 Member 데이터들을 가져오기 위해 단일 쿼리 100개를 추가로 날리게 됩니다. 즉 총 100 + 1 번의 쿼리가 날라가게 됩니다. 이러한 문제를 N+1 문제라고 합니다. 이러한 문제를 없애기 위해서 직접 지연 로딩으로 설정해야 합니다.
3. 컬렉션은 필드에서 초기화하는 것이 좋습니다. null 문제에 있어서 안전하기도 하고 하이버네이트는 엔티티를 영속화할 때, 컬렉션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경합니다. 만약 임의의 메서드에서 컬렉션을 잘못 생성하면 하이버네이트 내부 메커니즘에 문제가 발생할 수도 있기 때문에 필드 레벨에서 생성하는 것이 가장 안전하고 코드도 간결해지는 방법입니다.
4. 연관 관계에 cascade 옵션을 줄 수 있는데 cascade란 전파하는 것을 의미합니다. 예를들어서 Order 엔티티의 필드로 orderItems라는 리스트가 있을 때 Order 객체 1개 내 orderItems필드에 orderItemA, orderItemB, orderItemC를 데이터로 담아야 한다면 원래 JPA에서는 이를 위해 persist(orderItemA), persist(orderItemB), persist(orderItemC), persist(order)를 해줘야 합니다. 하지만 orderItems라는 필드 위에 @OneToMany(cascade = CascadeType.ALL)과 같은 옵션을 주면 전파되어 persist(order) 한번만으로 모두 DB에 저장 및 반영할 수 있습니다.
5. 양방향 연관 관계일 때 편리하게 양쪽 클래스 모두 다 한꺼번에 세팅하기 위해서 아래와 같은 코드를 사용하는 것이 좋습니다. 이렇게 하지 않으면 각각 클래스에 따로 필드를 세팅하는 메소드를 따로 만들어서 따로 각각 호출해야하는데 이렇게 묶어서 원자적인 함수를 만들면 편리하게 사용할 수 있습니다.   
   
``` java
    //연관 관계 메소드
    public void setMember(Member member) {
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem) {
        this.getOrderItems().add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery) {
        this.setDelivery(delivery);
        delivery.setOrder(this);
    }
```

## JPA를 활용한 MemberRepository 예시
> JPA를 활용하여 MemberRepository를 작성한 코드 예시입니다. 우선, @PersistenceContext 어노테이션을 통해 스프링이 시작될 때 만들어주는 EntityManager를 주입받습니다. 그 후 이 em을 활용해서 쿼리를 날리게 되는데 쿼리의 결과가 단일 객체인 경우에는 JPQL이 필요없지만 여러 객체로 리턴되는 경우에는 JPQL을 사용해 쿼리를 날리게 됩니다. JPQL은 테이블 대상이 아니고 엔티티를 대상으로 쿼리를 날린다고 생각하면 됩니다. 특히 where 문이 들어가는 쿼리를 날릴 때 파라미터 바인딩을 활용하게 되는데 아래 예제를 보면 **:name**이라고 된 부분이 다음 줄의 setParameter의 **"name"**과 바인딩 되어 **"name"**의 value가 전달되게 됩니다.   
   
```java
package jpabook.jpashop.repository;

import jpabook.jpashop.domain.Member;
import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import java.util.List;

@Repository
public class MemberRepository {

    @PersistenceContext
    private EntityManager em;

    public void save(Member member)  {
        em.persist(member);
    }

    public Member findOne(Long id) {
        return em.find(Member.class, id);
    }

    public List<Member> findAll() {
        //첫번째에 JPQL을 쓰고 두번째에 반환 타입을 적으면 된다. JPQL은 테이블 대상이 아니고 엔티티를 대상으로 쿼리를 날린다고 생각하면 됌
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findByName(String name) {
        return em.createQuery("select m from Member m where m.name = :name", Member.class)  //:name이라고 하면 밑에서 setParameter를 통해 "name"의 value와 바인딩 되는 것이다.
                .setParameter("name", name) //위에 있는 :name이 여기 파라미터의 key값과 바인딩되어서 value가 위로 넘어가게 된다.
                .getResultList();
    }
}
```

## JPA를 활용한 MemberService 예시
> JPA를 활용하여 MemberService를 작성한 코드 예시입니다. 아래 코드에서 사용되는 핵심 개념들을 살펴보겠습니다. 우선, JPA를 통해 데이터 변경을 가져다주는 모든 로직들은 트랙잭션 안에서 처리되어야 합니다. 그래서 우리는 @Transactional 어노테이션을 클래스 위에 붙여주었습니다. 이렇게 클래스에 붙게 되는 해당 어노테이션은 클래스 내부에 퍼블릭 메소드에도 각각 적용이 되게 됩니다. 이때 readOnly = true라는 옵션을 함께 준 것을 확인할 수 있는데 이렇게 되면 단순 조회 쿼리를 날릴 때 성능 최적화가 되어 성능이 좋아집니다. 하지만 해당 클래스 내 메소드 중에서 쓰기 쿼리를 날리는 메소드에 대해서는 아래와 같이 별도로 readOnly = false 값이 디폴트로 설정되어 있는 @Transactional 어노테이션을 따로 붙여줘야 합니다. 또한 아래 코드를 보시면 유효성 검사 메소드를 만들어줬는데, 웹 애플리케이션 서버가 여러 개 뜨는 멀티 스레드 환경에서 해당 유효성 검사 메소드를 동시에 통과할 가능성이 있는 경우에 문제가 발생할 수 있으므로 실무에서는 최후의 방어 목적으로 데이터 베이스 단에서 Member의 Name 컬럼의 제약조건을 UNIQUE로 반드시 설정해줘야 합니다.   
   
```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.repository.MemberRepository;
import lombok.AllArgsConstructor;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@Transactional(readOnly = true)  //JPA를 통한 데이터 변경 모든 로직들은 트랜잭션 안에서 처리되어야 한다. 이 어노테이션이 javax에서 제공하는 것과 spring에서 제공하는 것이 있는데
//spring에서 제공하는 걸 쓰는 것이 더 좋다. 왜냐하면 더 기능이 많다.
//그리고 클래스 위에 다는 트랜잭셔널 어노테이션은 기본적으로 클래스 내 들어있는 모든 public 메소드에 적용이 되는데 readOnly = false를 설정해주면 단순 조회 쿼리할 때 성능 최적화가 되어 좋다.
//현재 이 클래스에서는 단순 조회 쿼리 메소드가 더 많기 때문에 클래스 단에는 readOnly = true를 걸어주었다.
//이때 쓰기 쿼리를 날리는 메소드에 대해서는 아래와 같이 default 값으로 readOnly = false가 되어있는 @Transactional 어노테이션을 따로 붙여줘야 한다.
//@AllArgsConstructor //해당 클래스 내 모든 필드에 대한 생성자를 만들어 준다.
@RequiredArgsConstructor    //final이 붙은 필드에 대한 생성자만 만들어 준다.
public class MemberService {

    private final MemberRepository memberRepository; //final을 붙이면 const와 같아서 생성 시점에 반드시 생성자를 정의해서 초기화를 해줘야만하고 그 이후에 값 변경이 불가능하게 된다.
    //객체지향적 관점에서 데이터 변경 가능성이 적어져 더 안전하다

//    @Autowired  //DI(의존성 주입)를 해줄 때 setter, field, constructor 세 가지 방법 중 constructor 방식이 제일 좋다.
//    //그리고 참고로 요즘 스프링에서는 생성자가 하나 밖에 없는 경우에는 자동으로 @Autowired를 해주기 때문에 생략해도 된다.
//    public MemberService(MemberRepository memberRepository) {
//        this.memberRepository = memberRepository;
//    }

    //회원 가입
    @Transactional
    public Long join(Member member) {
        validateDuplicateMember(member);    //중복 회원 검증
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {   //사실 이 유효성 검사 메소드는 완벽하지 않다. WAS(Web Application Server)가 여러개 뜬 상태에서
        //멀티 스레드 환경에서 동시에 같은 이름으로 회원가입을 할 경우 해당 로직을 동시에 통과할 가능성이 있기 때문이다.
        //그래서 실무에서는 최후의 방어 목적으로 DataBase 단에서 Member의 Name 컬럼의 제약조건을 UNIQUE로 설정해주는 것이 좋다.
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if (!findMembers.isEmpty()) {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        }
    }

    //회원 전체 조회
    public List<Member> findMembers() {
        return memberRepository.findAll();
    }

    public Member findOne(Long memberId) {
        return memberRepository.findOne(memberId);
    }


}
```

## @AllArgsConstructor와 @RequiredArgsConstructor
> 위 MemberService 예시 코드를 보시면 @AllArgsConstructor와 @RequiredArgsConstructor를 사용한 부분이 있습니다. @AllArgsConstructor는 해당 클래스 내 모든 필드에 대한 생성자를 알아서 만들어주는 롬복이고, @RequiredArgsConstructor는 final이 붙은 필드에 대해서 생성자를 만들어줍니다. 여기서 final에 대해서 자세히 알 필요가 있습니다. final을 붙이면 마치 C++의 const와 같아서 해당 객체 생성 시점에 반드시 생성자를 정의해서 초기화를 해줘야만 하고 그 이후에는 값 변경이 불가능학 됩니다. 이렇게 사용하는 것이 객체지향 프로그래밍 관점에서 데이터 변경 가능성이 적어져 유지보수하기가 더 쉽고 안전하게 됩니다. 또한 이를 활용해서 아래와 같이 MemberRepository에 EntityManager를 Injection할 때 활용할 수도 있습니다. 원래는 EntityManager 필드를 만들고 생성자를 만들어서 거기에 @Autowired를 붙여 Injection해도 되지만 이를 final 필드와 @RequiredArgsConstructor 어노테이션만으로 간단하게 구현할 수도 있습니다. 사실 원래 스프링에서는 EntityManager를 injection 받으려면 @PersistenceContext가 있어야 하지만 spring boot가 @Autowired로 Injection 받을 수 있게 구현해주기 때문에 이것도 가능한 것입니다.   
   
```java
package jpabook.jpashop.repository;

import jpabook.jpashop.domain.Member;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import java.util.List;

@Repository
@RequiredArgsConstructor
public class MemberRepository {

//    @PersistenceContext
//    private EntityManager em;

    //위 EntityManager 코드를 아래와 같이 작성하는 것도 좋다. 일관성 목적
    //원래 스프링에서는 EntityManager를 injection 받으려면 @PersistenceContext가 있어야하지만
    //springboot가 @Autowired도 injection 받도록 지원을 해줌.
    //여기서 필드가 현재 하나만 있기 때문에 자동으로 @Autowired가 들어가서 injection이 가능한 것이다.
    private final EntityManager em;
}
```

## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard