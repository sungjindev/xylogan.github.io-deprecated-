---
title: 인프런 스프링 부트와 JPA 활용2 김영한 강의 정리 1
categories: [Computer engineering, Backend engineering]
tags: [백엔드, 스프링, 자바, backend, spring, java, JPA]
---

Backend engineering을 공부하는 학생이라면 한번쯤 들어봤을 [김영한 님의 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94/dashboard)를 들으며 기억해두면 도움될 만한 내용을 정리할 예정입니다. 강의를 모두 수강할 때까지 본 강의 정리 포스팅은 버전을 업데이트하며 주기적으로 업로드 할 예정입니다. 하지만, 주로 기록 목적의 포스팅이다보니 강의 목차 흐름에 따라 내용이 순차적으로 정리되어 다소 본문 내용의 일관성은 떨어질 수 있음을 미리 참고바랍니다. 

## MemberApiController 예시 코드
> JPA 활용1 강의에서 MVC 패턴으로 작성했던 코드를 요즘 트렌디한 RESTful API 방식으로 변경하기 위해 API 컨트롤러 예시 코드를 작성해보면 아래와 같습니다. **컨트롤러를 만들 때 가장 주의할 점으로 RequestBody와 RequestResponse에 절대 Entity를 그대로 노출해서는 안됩니다.** Entity가 그대로 노출되는 순간 보안 상의 이슈도 있지만 코드의 유지 보수성과 유연성 자체가 매우 크게 떨어지게 되고 Entity의 스펙이 바뀌는 순간 API의 스펙도 함께 변경되어야 하기 때문에 비즈니스 로직이 복잡한 실무에서는 더 큰 리스크로 다가오게 됩니다. 따라서 Data Transfer Object(DTO)를 사용하여 RequestBody와 RequestResponse를 별도의 내장 클래스로 따로 만들어주어야 합니다. Get 방식의 요청 같은 경우에는 별도의 요청 파라미터가 필요 없으므로 RequestBody에 매핑되는 DTO를 만들 필요가 없지만 Post 방식의 요청 API에는 Request DTO를 만들어 사용하였습니다. 이때 @RequestBody 어노테이션은 JSON으로 들어온 요청 바디를 모두 매핑해서 파라미터 객체에 맞게 넣어주는 역할을 합니다. 이와 추가적으로 중요한 점이 **List 형의 데이터를 그대로 Response로 보내지 말아야 합니다. List 형 그 자체로 ResponseBody를 보내게 되면 유연성이 매우 떨어지므로 제네릭 클래스를 별도로 만들어서 한번 객체로 감싸준 뒤에 보내주도록 해야합니다.**    
   
```java
package jpabook.jpashop.api;


import jpabook.jpashop.domain.Member;
import jpabook.jpashop.service.MemberService;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import javax.validation.constraints.NotEmpty;
import java.util.List;
import java.util.stream.Collectors;

// @Controller @ResponseBody   //@Controller와 @ResponseBody를 합친 것이 @RestController이다.
@RestController
@RequiredArgsConstructor
public class MemberApiController {

    private final MemberService memberService;

    @GetMapping("/api/v1/members")
    public List<Member> membersV1() {
        return  memberService.findMembers();
    }

    @GetMapping("/api/v2/members")
    public Result membersV2() {
        List<Member> findMembers = memberService.findMembers();
        List<MemberDto> collect = findMembers.stream()
                .map(m -> new MemberDto(m.getName()))
                .collect(Collectors.toList());

        return new Result(collect);
    }

    @Data
    @AllArgsConstructor
    static class Result<T> {
        private T data;
    }

    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String name;
    }


    @PostMapping("/api/v1/members")
    public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
        //@RequestBody가 하는 역할은 JSON으로 들어온 요청 바디를 모두 매핑해서 Member에 맞게 넣어준다.
        //근데 여기서 큰 문제점이 하나 있다. 보면 알겠지만 엔티티를 그대로 API의 파라미터로 사용하고 있는데
        //이렇게 되면 나중에 엔티티 스펙이 바뀌면 매핑된 API 스펙도 하나하나 다 바꿔줘야한다.
        //뿐만 아니라 지금 @Valid 옵션을 주고 Entity에서 모든 접근 제한 등을 처리하고 있는데
        //어떤 API에서는 다른 접근 제한이 필요할 수도 있다. 이런 것들이 문제가 되기 때문에
        //API 스펙을 위한 별도의 DTO(Data Transfer Object)를 만들어야 한다.
        //따라서 앞으로 API를 만들 때는 절대 엔티티를 파라미터로 받지 말자!!!
        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @PostMapping("/api/v2/members")
    public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
        Member member = new Member();
        member.setName(request.name);
        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @PutMapping("/api/v2/members/{id}")
    public UpdateMemberResponse updateMemberV2(@PathVariable("id") Long id, @RequestBody @Valid UpdateMemberRequest request) {
        memberService.update(id, request.getName());
        Member member = memberService.findOne(id);  //원래 위 udpate 메서드에서 바로 member 객체를 리턴해서 받아 써도 되지만
        //영한님은 유지보수성을 위해 커맨드와 쿼리(여기서는 조회쿼리)를 분리하는 방식을 선호한다고 함. 그래서 별도로 findOne()으로 조회 쿼리를 날린 것이다.
        return new UpdateMemberResponse(member.getName(), member.getId());
    }

    @Data
    static class CreateMemberRequest {
        @NotEmpty
        private String name;
    }


    @Data
    static class CreateMemberResponse {
        private Long id;

        public CreateMemberResponse(Long id) {
            this.id = id;
        }
    }

    @Data
    static class UpdateMemberRequest {
        private String name;
    }

    @Data
    @AllArgsConstructor //DTO에는 조금 롬복을 자유롭게 쓰는 편이라 함!
    static class UpdateMemberResponse {
        private String name;
        private Long id;
    }


}
```

## @XToOne(@ManyToOne과 @OneToOne) 이슈1
> @XToOne(@ManyToOne과 @OneToOne) 관계를 가질 때 관련된 API 컨트롤러의 성능 최적화를 어떻게 할 수 있을지에 대해서 알아보겠습니다. 우선 해당 연관 관계가 양방향 연관 관계로 이어져 있는 경우에는 JSON과 객체를 매핑시켜주는 잭슨 라이브러리(@RequestBody)가 실행될 때 연관 관계에 따라서 필드를 탐색하다가 양쪽 필드에 서로 참조하는 필드가 있으면 무한 루프에 빠지게 되므로 양방향 연관 관계를 가지고 있다면 어느 한쪽의 필드에는 @JsonIgnore 어노테이션을 반드시 붙여줘야 합니다. 

## @XToOne(@ManyToOne과 @OneToOne) 이슈2
> @XToOne에서 발생하는 문제점으로 API 컨트롤러에서 @ResponseBody를 사용하게 되는데 이 어노테이션을 사용하면 잭슨 라이브러리가 컨트롤러 내에서 리턴하는 객체를 JSON 형식으로 자동으로 변환해주는 역할을 하게 됩니다. 이때 @XToOne(fetch = LAZY)로 되어있다면 지연 로딩 로직 상 하이버네이트가 임시로 프록시 객체를 만들고 상속 받아서 해당 필드를 일단 채워넣습니다 . 이때 사용하는 라이브러리가 byteBuddy입니다. 이렇게 프록시를 이용해서 가짜 객체를 채워 넣은 다음에 해당 필드를 사용하게 될 때 그제서야 데이터베이스에 접근해서 정말 쿼리를 날리고 해당 프록시 객체의 값을 데이터베이스의 값을 불러와 채워넣게 됩니다. 이렇게 되면 잭슨 라이브러리가 해당 필드의 객체를 뽑아 쓰려고 할 때 byteBuddy에 의한 프록시 객체가 들어와 있어서 정상적으로 처리하지 못하고 에러가 발생하게 됩니다. 이를 해결하기 위해서는 **하이버네이트5 모듈**을 설치한 다음 잭슨 라이브러리한테 지연로딩하는 객체는 무시하고 뽑아달라고 요청할 수 있습니다. 우선 하이버네이트5 모듈을 설치하기 위해서 **implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'**를 build.gradle에 추가해줘야 합니다. 그 후 메인 애플리케이션 클래스에 하이버네이트5를 @Bean으로 만들어줍니다. 이러면 하이버네이트5 모듈이 잭슨에게 지연 로딩에 대해서는 무시하고 처리하라고 요청하게 됩니다. 근데 어찌됐건 이렇게 엔티티 자체를 노출해서 API를 만드는 것은 절대 하면 안됩니다!
 

## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94/dashboard

