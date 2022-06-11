---
title: gcloud Command-line interface Essentials
tags: [Cloud Computing, Google Cloud Platform]
style: fill
color: danger
description: gcloud CLI(Command-line interface) 입문자를 위한 기본 지식
---

[Google Cloud Platform Essentials 포스팅](https://l-o-g-a-n.github.io/blog/google-cloud-essentials)에 이어서 이번 포스팅에는 gcloud CLI에서 사용되는 기본적인 Command에 관하여 자세히 다뤄보려고 한다.

## Google Cloud Shell이란?
> Cloud Shell은 Google cloud 내 Resources에 대하여 Command-line 기반으로 사용할 수 있게 해준다. Cloud Shell은  
Debian 기반의 Virtual machine이고 5GB의 Home directory를 가지고 있다.  
Cloud Shell에는 gcloud Command-line tool 외에도 빠르고 쉬운 배포를 위한 다양한 툴들이 사전 설치되어 있다.  


## gcloud를 통해 Regions과 Zones 관리
> Regions과 Zones에 대한 개념은 이전 포스팅 [Google Cloud Platform Essentials 포스팅](https://l-o-g-a-n.github.io/blog/google-cloud-essentials)에서 충분히 알아보았다. 따라서, 이에 대한 설명은 생략하고 어떤 명령어를 통해 이들을 관리할 수 있는지 알아보겠다.

* Region 설정하기
```
gcloud config set compute/region us-central1
```

* 프로젝트의 Region 조회
```
gcloud config get-value compute/region
```

* Zone 설정하기
```
gcloud config set compute/zone us-central1-b
```

* 프로젝트의 Zone 조회
```
gcloud config get-value compute/zone
```

## gcloud를 통해 Project 정보 조회
> GCP 안에는 효율적인 관리를 위해 프로젝트 단위로 관리된다. 이러한 프로젝트는 각각 고유한 Project id를 가지고 있다.

*  Project id 조회
```
gcloud config get-value project
```

* Project에 대한 세부 정보 조회
```
gcloud compute project-info describe --project $(gcloud config get-value project)
```
위 Command에서 \$(명령어)은 앞으로도 자주 나올 문법이니 잘 알아두도록 하자!     
**\$(명령어) 구문은 안에 명령어를 실행한 결과를 그 자리에 내뱉는다.** 위 예시에서는 해당 명령어가 Project id를 리턴하는 것이다.