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

## CascadeType
> JPA를 활용해서 연관 관계를 줄 때 @OneToMany, @OneToOne 등의 어노테이션을 사용하게 됩니다. 이때 **@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)**와 같이 cascade 옵션을 주게되면 해당 필드가 선언되어있는 엔티티가 persist될 때 cascade 옵션이 걸려있는 필드들도 연쇄적으로 함께 persist되게 됩니다. 따라서 아래 코드에서처럼 **orderRepository.save(order);**로 order만 persist해줘도 order 엔티티 안에 cascade 옵션이 걸려있는 orderItems, delivery 필드들도 함께 persist되어 별도로 개별 repository를 만들어서 persist를 호출할 필요가 없습니다.   
   
```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Delivery;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderItem;
import jpabook.jpashop.domain.item.Item;
import jpabook.jpashop.repository.ItemRepository;
import jpabook.jpashop.repository.MemberRepository;
import jpabook.jpashop.repository.OrderRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final MemberRepository memberRepository;
    private final ItemRepository itemRepository;

    /*
    주문
     */
    @Transactional
    public Long order(Long memberId, Long itemId, int count) {

        //엔티티 조회
        Member member = memberRepository.findOne(memberId);
        Item item = itemRepository.findOne(itemId);

        //배송정보 생성
        Delivery delivery = new Delivery();
        delivery.setAddress(member.getAddress());

        //주문 상품
        OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);

        //주문 생성
        Order order = Order.createOrder(member, delivery, orderItem);

        //주문 저장
        orderRepository.save(order);
        return order.getId();

    }
}
```

## DDD를 위해 Entity에 별도의 생성 메소드를 만들어 협업할 때
> Domain Driven Design(DDD)을 위해 Entity에 별도의 생성 메소드를 만들었는데 다른 사람과 협업 시에 어떤 사람은 해당 생성 메소드를 잘 활용하는 반면 어떤 사람들은 기본 생성자를 활용해 코드를 짜다보면 나중에 유지보수하기가 어려워집니다. 이럴 때 제약 조건들을 걸어서 프로그래밍을하면 다른 사람과 협업 시에도 코드를 통일성있게 짤 수 있어 도움이 됩니다. 이렇게 하기 위해서는 제가 만든 별도의 생성 메소드를 통해서만 해당 엔티티를 생성할 수 있도록 해야하는데, 그땐 아래와 같이 protected 접근 지정자를 가지는 기본 생성자를 만들어 놓으면 됩니다. 또한 아래 코드는 롬복으로 줄일 수 있는데 기본 생성자를 protected 접근 지정자로 설정하고 싶으면 해당 클래스 위에 **@NoArgsConstructor(access = AccessLevel.PROTECTED)**로 설정하면 됩니다. 이렇게하면 다른 위치에서 기본 생성자를 통해 엔티티를 생성하려고 하면 IDE 상에서 에러가 뜨게 됩니다.
   
```java
    protected OrderItem() {
    }
```

## 컨트롤러에서 Validation
> 컨트롤러에서 넘겨받는 객체에 대해 Validation을 할 수 있습니다. 우선 넘겨받는 객체 클래스에서 Validation하고 싶은 필드에 **@NotEmpty** 등과 같은 어노테이션을 붙여 준 다음 해당 객체를 컨트롤러에서 넘겨받을 때 파라미터 앞에 **@Valid**라고 붙여주게 되면 컨트롤러에서 javax의 Validation을 사용하는구나 라고 인지한 뒤에 validation을 해주게 됩니다. 아래 예시에서는 MemberForm 클래스 내 name 필드에 @NotEmpty를 넣었기 때문에 name 필드에 대한 유효성 검사를 진행하게 됩니다. 이때 사용자가 name 값을 넣지 않은 채로 post 요청을 보내게 되면 validation에 의해 에러를 가진 채로 해당 컨트롤러가 끝나게 되는데 아래 코드 예시처럼 BindingResult 객체를 만들어주면 에러가 발생했을 때 해당 에러를 BindingResult 객체에 가진 채로 정상적으로 해당 컨트롤러 바디로 진입하게 됩니다. 그래서 컨트롤러 바디 내에서 에러를 검출해서 원하는 페이지로 다시 보낼 수 있습니다. 아래 예시에서는 createMemberForm 페이지로 보내게 되는데 이때 스프링이 BingdingResult를 페이지로 함께 넘겨주게 되어 thymeleaf 같은 뷰 페이지에서 이를 활용해서 쓸 수 있게 됩니다.   
   
```java
package jpabook.jpashop.controller;

import lombok.Getter;
import lombok.Setter;

import javax.validation.constraints.NotEmpty;

@Getter @Setter
public class MemberForm {

    @NotEmpty(message = "회원 이름은 필수입니다.")    //이게 있으면 아래 name이라는 필드의 값이 비어있으면 오류가 남
    private String name;

    private String city;
    private String street;
    private String zipcode;


}
```   
   
```java
package jpabook.jpashop.controller;

import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.service.MemberService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

import javax.validation.Valid;

@Controller
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    @PostMapping("/members/new")
    public String create(@Valid MemberForm form, BindingResult result) {  //@Valid 어노테이션을 붙이면 javax의 validation을 쓰는구나하고 인지한 뒤
        //validation을 해주게 됌. 우리는 MemberForm에 @NotEmpty를 넣었기 때문에 그 유효성 검사를 쓸 수 있음
        //이렇게 validation에 걸리게되면 원래 에러를 갖고 이 컨트롤러 밖으로 나가버리는데 BindingResult 객체를 하나 만들어주면
        //에러가 발생했을 때 그 에러를 BindingResult 객체에 담고 정상적으로 이 컨트롤러 바디로 진입하게 됌!

        if (result.hasErrors()) {   //그래서 이렇게 에러를 검출해서 원하는 페이지로 보내버릴 수도 있음!!!
            return "/members/createMemberForm"; //이때 createMemberForm 페이지로 넘어갈 때 스프링이 BindingResult를 함께 보내줌
        }   //그래서 thymeleaf가 뷰 페이지에서 이를 활용해서 쓸 수 있음!

        Address address = new Address(form.getCity(), form.getStreet(), form.getZipcode());

        Member member = new Member();
        member.setName(form.getName());
        member.setAddress(address);

        memberService.join(member);
        return "redirect:/";
    }
}
```

## Form 클래스 생성
> JPA를 활용해서 개발을 하다보면 언제는 Entity 그 자체를 사용하고 언제는 별도의 Form 클래스를 만들어서 사용합니다. 가급적이면 Entity와 Form은 분리해주는 것이 좋고 서버 쪽에서 자체적으로 뷰 템플릿을 꾸려 내부적으로만 사용할 때는 Entity를 일부 출력 목적으로 사용할 수도 있겠지만 **API를 통해 객체를 반환해야할 경우에는 절대로 Entity 그 자체를 반환해서는 안됩니다.**

## 엔티티 업데이트 시 변경 감지 혹은 Merge
> 엔티티의 값을 업데이트한 뒤 데이터베이스에 반영하고 싶을 때 두가지 방법을 사용할 수 있습니다. 하나는 Merge라는 방법과 또 다른 하나는 JPA 영속성 컨텍스트가 제공하는 변경 감지, 일명 Dirty Checking을 활용하는 방법이 있습니다. 결론부터 말하자면 Merge 방식은 절대 사용하지 말고 값 변경 감지에 의한 Dirty Checking을 사용할 것을 권장합니다. 우선 아래 코드가 대표적인 Merge의 예시입니다. BookForm이라는 클래스의 객체 값을 파라미터로 넘겨받은 뒤 book라는 새로운 객체를 만들어주고 새로 만든 객체에 값을 덮어 씌워 그 객체를 데이터베이스에 저장하려고 시도하고 있습니다. 아래 코드에는 나와 있지 않지만 itermService.saveItem()을 더 타고 들어가면 내부적으로 Merge()를 호출하여 처리하고 있습니다. 이렇게 새로 만든 객체는 JPA에서 관리하는 영속성 객체에 해당하지 않지만 영속성 객체의 id값은 그대로 지니고 있으므로 준영속 엔티티라고 부릅니다. 이러한 준영속 엔티티는 영속 엔티티와 다르게 JPA에 의해 관리되지 않아 Dirty Checking이 되지 않습니다. 따라서 Merge를 활용하게 되는데 Merge란 넘겨받은 준영속 엔티티의 각각의 필드 값으로 관련된 영속 엔티티의 값을 모두 대체(머지)시켜 버리고 영속 엔티티 값을 리턴하는 메서드입니다. 이렇게 되면 준영속 엔티티에 의도하지 않은 NULL 값 등이 필드에 들어가있을 때 강제로 영속 엔티티의 값에도 NULL이 들어가게 되어 데이터베이스에 그대로 반영됩니다. 따라서, Merge를 사용하기 보다는 무조건 Dirty Checking을 활용하는 것이 좋습니다. Dirty Checking을 활용한다는 것은 form.getId() 등과 같은 메서드를 활용해서 직접 itemService에서 영속성 객체를 조회한다음 해당 객체에 setter나 비즈니스 메서드 등을 활용해서 값을 설정해주는 것입니다.   
   
```java
    @PostMapping("items/{itemId}/edit")  //여기서 {itemId}는 path variable이다.
    public String updateItem(@PathVariable String itemId, BookForm form) {

        Book book = new Book(); //이것 같은 경우에는 새로운 객체를 생성해서 값을 넣고 있고 그걸 레포지토리에 저장하고 있다.
        //이러면 이건 영속성 관리가 되고 있는 객체가 아니라서 준영속 엔티티라고 부르게 되는데 이런 엔티티들을 Merge()로 저장하게 되면
        //선택 할 수 없이 모든 필드가 준영속 엔티티의 필드 값으로 강제로 머지(대체) 되버린다. 이러면 의도치 않은 새로운 객체의 NULL값 등이 들어갈 수 있다.
        // 따라서 이걸 쓰기보단 무조건적으로 JPA 영속성 컨텍스트가 제공하는 변경 감지, 즉 Dirty Checking을 활용해야한다.
        //여기서 예를들면 form.getId()로 직접 itemService를 활용해서 영속성 객체를 조회한다음 거기에 setter를 써주면 된다.
        //근데 사실 setter보다도 Entity 내에 비즈니스 메소드를 만들어서 쓰는게 좋다. Domain Driven Design으로 !!!
        book.setId(form.getId());
        book.setName(form.getName());
        book.setPrice(form.getPrice());
        book.setStockQuantity(form.getStockQuantity());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());

        itemService.saveItem(book);
        return "redirect:/items";
    }
```

## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard