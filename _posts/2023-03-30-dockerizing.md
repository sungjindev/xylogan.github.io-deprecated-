---
title: Dockerizing Spring Boot application
categories: [Computer engineering, Cloud infrastructure]
tags: [infrastructure, docker, dockerizing, spring, 인프라, 도커, 도커라이징, 스프링]
---

현재 저는 Spring Boot + JPA를 활용해서 백엔드 애플리케이션을 개발하고 있으며 이번 포스팅에서는 Spring Boot 애플리케이션을 Dockerizing하는 과정과 그 과정에서 겪었던 이슈들에 대해 적어보려고 합니다.

## Docker란?
Docker는 컨테이너를 기반으로 하는 오픈소스 가상화 플랫폼입니다. 쉽게 말해 컨테이너 기반으로 어플리케이션을 작성하고 동작할 수 있게 해줍니다. 이렇게 만들어진 컨테이너는 시스템 환경에 구애받지 않고 도커 엔진만 있다면 어디서든 동일하게 실행시킬 수 있습니다. 이것이 컨테이너의 큰 장점입니다.

## Dockerizing
저는 현재 개발하고 있는 Spring Boot + JPA 기반의 백엔드 애플리케이션을 Google Cloud Platform(GCP)에 배포할 계획을 가지고 있으며, 그 전에 앞서 로컬에서 컨테이너화된 상태로 간단히 테스트할 수 있게 Dockerizing을 진행하였습니다. 소스 코드를 보면서 어떻게 컨테이너화 했는지 알아보겠습니다.

## Dockerfile
```
FROM openjdk:11-jdk

RUN apt-get update && apt-get -y install sudo

COPY iampotato-*.jar app.jar

ENV PROFILE dev

ENTRYPOINT ["java", "-Dspring.profiles.active=${PROFILE}", "-jar", "/app.jar"]
```   
    
우선 Spring Boot application을 도커라이징하기 위해 만든 Dockerfile입니다. 저는 해당 파일을 프로젝트 내 **build/libs** 경로에 만들어줬습니다. 이렇게 한 이유는 spring application을 띄우기 위해 bootjar 파일들이 필요한데 이를 얻기 위해 빌드하면 해당 경로에 생성되고, 도커 파일 내부에서 이러한 파일들을 COPY 하기 위해 접근할 때 도커 파일이 위치한 경로가 홈 디렉토리가 되기 때문에 편리하게 접근하기 위해서 위와 같은 경로에 Dockerfile을 생성해줬습니다.   
위 코드에서 중요한 내용을 설명드리자면 **ENV PROFILE dev**를 통해서 프로파일을 dev로 설정하였는데 이렇게 설정된 프로파일에 따라 application 설정 파일을 다르게 실행할 수 있습니다. 위처럼 저는 dev로 설정하였기 때문에 application-dev.yml 파일에 정의된 설정대로 애플리케이션이 실행되게 됩니다.

## docker-compose.yml
```
version: '3'

services:
  database-mysql:
    container_name: database-mysql
    image: mysql/mysql-server:8.0

    environment:
      MYSQL_ROOT_PASSWORD: '비밀번호를 입력하세요'
      MYSQL_ROOT_HOST: '%'
      MYSQL_DATABASE: 'iampotato'
      TZ: Asia/Seoul

    volumes:
      - ./mysql-init.d:/docker-entrypoint-initdb.d

    ports:
      - '13306:3306'

    command:
      - '--character-set-server=utf8mb4'
      - '--collation-server=utf8mb4_unicode_ci'

  database-adminer:
    container_name: database-adminer
    image: adminer:latest
    ports:
      - "18080:8080"
    environment:
      - ADMINER_DEFAULT_SERVER=database-mysql
      - ADMINER_DESIGN=hydra
      - ADMINER_PLUGINS=tables-filter tinymce

  application:
    container_name: app-iampotato
    build: ./
    restart: always
    ports:
      - '80:8080'

    depends_on:
      - database-mysql
```   
   
docker-compose.yml 파일은 여러 개의 컨테이너를 동시에 띄워 의존 관계 등도 설정해주며 Container Orchestration 할 때 사용합니다. 저는 Spring Boot 백엔드 applicaton과 MySQL 데이터베이스를 함께 사용하기 위해 각각 개별 컨테이너로 띄워 Orchestration 해줬습니다. 또한 추가적으로 adminer라는 DB관련 GUI 웹 서버도 함께 띄워 실행시켜줬습니다.   
여기서도 핵심적인 내용을 위주로 설명드리면, 도커 컴포즈 같은 경우에는 도커 컨테이너 간 내부 네트워크 및 포트를 통해서 통신하고 이때 사용되는 것들이 container_name입니다. 위에서 볼 수 있듯이 depends_on 옵션을 걸어줄 때도 사용되며 추후 application-dev.yml에서도 사용되니 반드시 잘 기억해두시길 바랍니다.   
또 다른 중요한 부분으로, 위에서 volumnes를 통해 프로젝트 내 mysql-init.d 디렉터리와 docker-entrypoint-initdb.d를 마운트시켜줬습니다. 따라서 도커가 처음 시작될 때 해당 mysql-init.d 디렉터리 내 sql 파일들이 실행되게 됩니다. 저와 같은 경우에는 아래 보시는 것처럼 database를 새로 생성해서 초기화해주는 쿼리를 작성하였습니다. 마지막으로 위 코드에서 반드시 알아둬야 하는 옵션으로 build가 있습니다. 해당 옵션은 Dockerfile을 통해 이미지를 빌드할 경우 사용하는 옵션입니다. 현재 보고있는 docker-compose.yml 파일이 위치한 경로를 기준으로 빌드할 Dockerfile이 위치하는 경로에 맞게 상대 경로로 작성해주면 됩니다.

## mysql-init.d/init.sql
```sql
CREATE DATABASE iampotato DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```   
    
앞서 설명드린 것처럼 도커가 실행될 때 위 디렉토리 내 sql 파일들이 실행될 수 있도록 마운트시켜 놓았습니다. 위와 같은 쿼리가 도커가 처음 시작될 때마다 실행되면서 database를 새로 생성하여 초기화시켜주게 됩니다.

## application-dev.yml
```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://database-mysql:3306/iampotato
    username: root
    password: "비밀번호입니다."
```   
   
앞선 도커 파일에서 프로파일을 dev로 주면서 application-dev.yml 파일을 기본 애플리케이션 설정 파일로 사용하여 실행하게 되었습니다. 그러므로 url 또한 역시 현재 docker-compose를 통해 여러 컨테이너를 orchestration하여 띄우고 있으므로 반드시 container-name에 맞게 수정해줘야 합니다. 

## .gitignore 이슈
```
build/**
!build/
!build/libs
!build/libs/docker-compose.yml
!build/libs/mysql-init.d/
!build/libs/mysql-init.d/init.sql
!build/libs/Dockerfile
```   
도커와는 관련이 없지만 도커 관련 파일들을 프로젝트 내 build/libs에 위치시키면서 발생했던 .gitignore 관련 이슈를 적어보려고 합니다. 보통 일반적으로 스프링 부트 프로젝트의 .gitignore라면 build 내 모든 파일들을 ignore 시켜 관리하기 마련인데 저는 다른 백엔드 개발자와 협업하면서 도커 파일을 build 경로에 위치시켜 깃허브로 공유하고 싶었습니다.   
이를 쉽게 처리하기 위해 build를 ignore 시켜놓고 .gitignore에서 제공하는 기능으로 **!**를 예외 처리하고싶은 파일 앞에 붙여 적게되면 해당 파일이 예외처리가 되게 되는데, 해당 방식을 따라 작성해봤지만 정상적으로 동작하지 않았습니다. 그래서 .gitignore 관련 docs를 찾아보니 성능적인 이슈가 있어서 예외 처리하고자 하는 파일의 부모 디렉터리가 ignore 되어있으면 **!**를 통해 예외 처리를 하더라도 정상적으로 예외로 동작하지 않는다는 글을 보았고 이를 해결하기 위해서 위 코드와 같이 예외 처리하고자 하는 파일의 모든 부모 디렉터리를 예외처리 해주어 해결할 수 있었습니다.   



## References
> https://yainii.tistory.com/31   
https://velog.io/@gillog/Git-.gitignore-%ED%8A%B9%EC%A0%95-%ED%8C%8C%EC%9D%BC-%ED%8F%AC%ED%95%A8-%EC%8B%9C%ED%82%A4%EA%B8%B0
