---
title: 인프런 스프링 부트와 JPA 활용1 김영한 강의 정리 1
categories: [Computer engineering, Backend engineering]
tags: [백엔드, 스프링, 자바, backend, spring, java, JPA]
---

Backend engineering을 공부하는 학생이라면 한번쯤 들어봤을 [김영한 님의 실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard)를 들으며 기억해두면 도움될 만한 내용을 정리할 예정입니다. 강의를 모두 수강할 때까지 본 강의 정리 포스팅은 버전을 업데이트하며 주기적으로 업로드 할 예정입니다. 하지만, 주로 기록 목적의 포스팅이다보니 강의 목차 흐름에 따라 내용이 순차적으로 정리되어 다소 본문 내용의 일관성은 떨어질 수 있음을 미리 참고바랍니다. 

## H2 데이터베이스 설정
> 우선 미리 설치해둔 H2 데이터베이스에 연결하기 위해서 application.properties 등을 설정해줘야 하는데 application.yml을 생성해서 설정해줘도 됩니다. 영한 님께서는 yml 파일로 강의하시므로 저도 해당 파일로 설정하도록 하겠습니다. 코드는 아래와 같습니다. 아래 코드를 간단히 보면 데이터베이스 connection과 관련된 spring.datasource를 설정해주기 위해 아래와 같은 설정을 해주고 있습니다. 그 후 JPA와 관련된 설정을 보면 ddl-auto의 create 모드는 자동으로 테이블을 만들어주는 모드입니다. 특히, 애플리케이션 실행 시점에 테이블을 모두 한번 지운다음 다시 만들어줍니다. 마지막으로 로그 레벨을 정하는 설정도 들어가 있는데 org.hibernate.SQL을 debug 모드로 쓰면 JPA 하이버네이트가 생성하는 모든 SQL이 로거를 통해 남게되며 이와 똑같이 show_sql도 JPA 하이버네이트가 생성하는 모든 SQL을 보여주지만 이것은 로거를 통해 로그를 찍는 것이 아닌 Systems.out에 찍게 됩니다. 따라서 show_sql 사용을 지양하고 로거를 사용하도록 합니다.   
    
```yml
spring:
    datasource:
      url: jdbc:h2:tcp://localhost/~/jpashop
      username: sa
      password:
      driver-class-name: org.h2.Driver

    jpa:
      hibernate:
        ddl-auto: create #애플리케이션 실행 시점에 내가 가지고 있는 테이블을 다 지운 다음에 엔티티 정보를 보고 테이블을 자동으로 만들어 줌
      properties:
        hibernate:
#          show_sql: true #밑에 org.hivernate.SQL과 겹치는데 얘는 System.out으로 찍는 거고 밑에는 로그로 찍는거. 얘를 안쓰는 걸 추천
          format_sql: true

logging: #로그 레벨을 정하는 것
  level:
    org.hibernate.SQL: debug #JPA 하이버네이트가 생성하는 SQL이 다 보인다
```

## Live Templates
> IntelliJ Settings에 들어가서 Live Templates를 검색해보면 해당 탭에서 사용자가 자주 반복해서 쓰는 코드를 템플릿화 시키는 설정을 할 수 있습니다. 강의 중에서는 Test 코드를 작성할 때 //given, //when, //then을 많이 활용하므로 이와 관련된 아래 코드를 tdd라는 Abbreviation으로 만들어서 사용하고 있는데 이를 작성해두면 편리하므로 설정해두도록 합니다. Template text 코드는 아래와 같습니다.   
    
```java
@Test
public void $NAME$() throws Exception {
    //given
    $END$
    //when
    
    //then
}
```

## @Transactional 어노테이션
> JPA를 사용할 때 EntityManager를 사용하게 됩니다. EntityManager를 통한 모든 데이터 변경은 항상 트랜잭션 안에서 이루어져야 합니다. 따라서 테스트 코드에서 EntityManager를 통한 레포지토리 테스트를 한다던지 할 때 반드시 @Transactional 어노테이션을 붙여줘야 합니다. 그리고 특수한 경우로 Test 코드에서 @Transactional 어노테이션을 붙이면 자동으로 테스트가 끝난 뒤 테이블을 롤백시켜줍니다. 이래야 반복적인 테스트가 가능하기 때문입니다. 만약에 이 롤백 옵션을 끄고 확인하고 싶다면 @Rollback(false) 어노테이션을 붙이면 됩니다. testMember() 예제 코드는 아래와 같습니다.    
    
```java
package jpabook.jpashop;

import org.assertj.core.api.Assertions;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)    //Junit한테 스프링과 관련된 테스트를 한다고 알려주는 것
@SpringBootTest //Spring Boot로 테스트하기 위해 필요
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    @Transactional
    public void testMember() throws Exception {
        //given
        Member member = new Member();
        member.setUsername("memberA");

        //when
        Long saveId = memberRepository.save(member);
        Member findMember = memberRepository.find(saveId);

        //then
        Assertions.assertThat(findMember.getId()).isEqualTo(member.getId());
        Assertions.assertThat(findMember.getUsername()).isEqualTo(member.getUsername());

    }
}
```

## JPA 상속 관계 매핑
> 객체끼리는 상속이라는 개념이 존재하지만, RDB 내 테이블끼리는 상속이라는 개념이 없습니다. 그대신 데이터베이스의 슈퍼 타입, 서브 타입이라는 관계를 이용해 상속 관계를 매핑할 수 있습니다. 먼저, JPA에서 상속을 하려면 상속 관계 전략을 부모 클래스에 반드시 설정해줘야 합니다. InheritanceType에는 3가지가 있습니다. JOINED는 가장 정규화된 상태로 테이블을 만드는 것이고 SINGLE_TABLE은 한 테이블만 만들고 그 안에 모든 필드 데이터를 집어넣는 것을 말하며 마지막으로 TABLE_PER_CLASS는 각 클래스 별로 테이블을 모두 만드는 것을 말합니다. 본 강의에서는 SINGLE_TABLE 전략을 사용하고 있기 때문에 부모 클래스가 되는 Item 클래스에 **@Inheritance(strategy = InheritanceType.SINGLE_TABLE)**을 붙여줬습니다. 또한 SINGLE_TABLE 전략은 여러 클래스의 모든 필드가 하나의 테이블에 함께 만들어지기 때문에 이를 구분하기 위한 컬럼이 필요한데 이를 위해서 **@DiscriminatorColumn(name = "dtype")** 어노테이션을 통해 dtype이라는 컬럼의 구분용 컬럼을 만들어 줬습니다. 이렇게 한 뒤 상속을 받는 클래스에 가서 **@DiscriminatorValue("A")** 이런 식으로 상속 받는 클래스를 해당 컬럼에서 어떤 값으로 표현할지 설정해줘야 합니다. 예제 코드는 아래와 같습니다. Album이라는 클래스가 Item을 상속받는 예제입니다.   
    
```java
package jpabook.jpashop.domain.item;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;

@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
//상속을 하려면 상속 관계 전략을 부모 클래스에 반드시 잡아줘야 한다. JOINED는 가장 정규화된 상태로 하는 것, SINGLE_TABLE은 한 테이블에 다 합치는 거, TABLE_PER_CLASS 는 클래스 별로 테이블 나누는 것
@DiscriminatorColumn(name = "dtype")    //싱글 테이블 전략이다 보니까 서로 다른 Book, Album, Movie끼리 구분할 수 있게 컬럼을 만들어주는 것
@Getter @Setter
public class Item {

    @Id @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;
}
```    
    
```java
package jpabook.jpashop.domain.item;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.DiscriminatorValue;
import javax.persistence.Entity;

@Entity
@Getter @Setter
@DiscriminatorValue("A")
public class Album extends Item {
    private String artist;
    private String etc;

}
```

## JPA 내장 타입
> JPA에서 새로운 값 타입을 정의해서 사용할 수 있는데 이것을 내장 타입이라고 합니다. 사용 방법은 간단합니다. 내장 타입으로 사용할 클래스에 **@Embeddable** 어노테이션을 붙여주고 내장 타입을 사용하는 각 필드 위에는 **@Embedded** 어노테이션을 붙여주면 됩니다. 

## JPA 연관 관계
> JPA에서 RDB 연관 관계를 매핑해주기 위해서 1:N이면 @ManyToOne, @OneToMany를 사용하고 1:1을 매핑해주기 위해서 @OneToOne 등을 사용합니다. 관계가 매핑이 된다고 하더라도 양방향 연관 관계가 있을 수 있고 단방향 연관 관계가 있을 수 있는데 양방향 연관 관계에서는 받으시 연관 관계의 주인을 정해줘야 합니다. 왜냐하면 두 테이블 각각에서 모두 데이터 수정이 가능한 상태이므로 JPA가 모호하게 받아들이기 때문입니다. 연관 관계의 주인은 보통 FK(N이 되는 쪽)를 가지고 있는 클래스에 잡아주면 됩니다. 이를 위해서는 주인이 아닌 클래스에 가서 @OneToMany(mappedBy = "member")과 같이 mappedBy와 관련된 어노테이션을 붙여주면 됩니다. mappedBy란 여기서 주인 클래스 내에 있는 "member"라는 필드의 거울 일 뿐이라고 선언하는 것입니다. 즉 이렇게 어노테이션을 붙이면 여긴 읽기 전용이 되는 것이라서 여기서 값을 넣어도 FK 값이 변경되지 않게됩니다. 이때 추가적으로 1:1 관계 일 때는 FK를 어느쪽이 가져도, 즉 연관 관계의 주인이 어느 쪽이 되어도 상관이 없는데 일반적으로 비즈니스 로직상 더 많이 접근할 것 같은 곳에 주인을 잡아주면 됩니다. 다음 예제 코드는 1:N 관계를 가지는 Member와 Order 클래스입니다.   
   
```java
package jpabook.jpashop.domain;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Getter @Setter
public class Member {
    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;

    @Embedded   //내장 타입을 표함하고 있다는 것을 나타내줘야함
    private Address address;

    @OneToMany(mappedBy = "member")  //Member 엔티티 기준에서 Order의 list들이 1:N 이므로
    //양방향 연관 관계에서 연관 관계의 주인이 아니므로, mappedBy를 해주는데 여기서 member는 Order 테이블에 있는 FK인 member 필드를 의미함
    //Order 테이블 내 FK인 member 필드의 거울일 뿐이라고 선언하는 것, 즉 여기는 그냥 읽기 전용이 되는 것이라 여기서 값을 넣어도 FK 값이 변경되지 않음
    private List<Order> orders = new ArrayList<>();


}
```   
    
```java
package jpabook.jpashop.domain;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders") //SQL의 ORDER와 충돌이 날 수도 있어서 관례적으로 orders라는 테이블 이름으로 바꿔줘야함
@Getter @Setter
public class Order {
    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne  //Order 엔티티 기준에서 Order가 N이고 Member가 1이므로
    @JoinColumn(name = "member_id") //FK 이름이 member_id가 된다.
    //양방향 연관 관계를 가지고 있으므로 연관 관계의 주인을 잡아줘야한다. 왜냐하면 두 군데 각각에서 모두 연관 관계 변경이 가능한 상태이므로 JPA가 혼란스러워함.
    //연관 관계의 주인은 FK를 가지고 있는 얘로 잡으면 된다. 주인을 잡기 위해서는 주인아 아닌 곳에 가서 mapped by를 해주면 된다.
    private Member member;


    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne   //1:1 관계에서 연관 관계의 주인을 잡아주기가 되게 애매할 수 있는데 어느 곳을 잡아도 상관없지만 일반적으로 더 많이 접근할 것 같은 곳에 주인을 잡아주면 된다.
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    private LocalDateTime orderDate;    //주문 시간

    @Enumerated(EnumType.STRING)    //enum 타입을 쓸 때 반드시 붙여주기!
    //EnumType이 ORDINAL과 STRING이 있는데 ORDINAL은 enum 순서대로 1,2,3 이런식으로 숫자가 잡혀
    // 나중에 enum이 중간에 추가되면 db가 꼬이므로 STRING 쓰기
    private OrderStatus status; //주문 상태를 나타내는 enum 타입 [ORDER, CANCEL]


}
```

## JPA enum 타입
> JPA에서 enum 타입을 사용할 때는 반드시 enum 타입을 사용하는 필드 위에 **@Enumerated(EnumType.STRING)**과 같은 어노테이션을 붙여줘야 합니다. EnumType에는 ORDINAL과 STRING이 있는데 ORDINAL은 enum 순서대로 1,2,3 등과 같은 숫자가 매핑이 되는데 나중에 해당 enum 원소 중간에 데이터들이 추가되면 해당 순서가 틀어져 db가 꼬이므로 반드시 STRING을 사용해줘야 합니다. enum을 사용한 예제 코드는 아래와 같습니다.   
   
```java
package jpabook.jpashop.domain;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;

@Entity
@Getter @Setter
public class Delivery {

    @Id @GeneratedValue
    @Column(name = "delivery_id")
    private Long id;

    @OneToOne(mappedBy = "delivery")
    private Order order;

    @Embedded
    private Address address;

    @Enumerated(EnumType.STRING)
    private DeliveryStatus status;  //READY, COMP
}
```


## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard