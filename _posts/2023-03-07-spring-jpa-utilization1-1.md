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
      url: jdbc:h2:tcp://localhost/~/jpashop;MVCC=TRUE
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


## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard