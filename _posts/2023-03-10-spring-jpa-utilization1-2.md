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


## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard