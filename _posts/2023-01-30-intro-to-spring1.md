---
title: 인프런 스프링 입문 김영한 강의 정리 #1
categories: [Computer engineering, Backend engineering]
tags: [백엔드, 스프링, 자바, backend, spring, java]
---

Backend engineering을 공부하는 학생이라면 한번쯤 들어봤을 [김영한 님의 인프런 스프링 입문 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard)를 들으며 기억해두면 도움될 만한 내용을 정리할 예정입니다. 강의를 모두 수강할 때까지 본 강의 정리 포스팅은 버전을 업데이트하며 주기적으로 업로드 할 예정입니다. 하지만, 주로 기록 목적의 포스팅이다보니 강의 목차 흐름에 따라 내용이 순차적으로 정리되어 다소 본문 내용의 일관성은 떨어질 수 있음을 미리 참고바랍니다. 

## Spring 프로젝트 시작
> Spring 프로젝트를 시작하기 전에 앞서, 본 강의에서는 Java 11 버전을 사용하며, 개발을 위해 IntelliJ IDE를 사용합니다. 기본적인 설정을 모두 마친 뒤, 스프링을 통해 웹 서버 개발을 하기 위해 필요한 라이브러리 들을 불러올 설정들을 입력해줘야 하는데 요즘은 [spring initializr](https://start.spring.io/)를 사용해 쉽게 Web UI를 사용하여 사용자가 커스텀한 설정이 들어간 project 뼈대를 다운로드하여 활용할 수 있습니다. 저는 Java 11 버전을 사용하기 때문에 아래와 같이 설정해 줬습니다.
![1](/assets/img/intro_to_spring/#1/1.png){: w="50%" h="50%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

## References
> https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard