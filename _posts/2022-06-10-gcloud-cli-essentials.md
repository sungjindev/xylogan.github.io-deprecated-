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


## gcloud를 통해 환경 변수 설정
> Environment variable을 설정해두면 API나 실행 파일들을 포함하는 스크립트를 작성할 때 시간을 절약할 수 있게 해준다.

* Project id에 관한 환경 변수 설정
```
export PROJECT_ID=$(gcloud config get-value project)
```
앞서 배웠던 \$(명령어) 구문을 활용해서 PROJECT_ID라는 이름의 환경 변수에 현재 프로젝트의 id를 저장하였다.

* Zone에 관한 환경 변수 설정
```
export ZONE=$(gcloud config get-value compute/zone)
```

* 설정한 환경 변수 조회
```
echo -e "PROJECT ID: $PROJECT_ID\nZONE: $ZONE"
```

## gcloud를 통해 Virtual machine 생성
> 이전 포스팅을 보고 온 사람이라면 아래 명령어가 낯 익어보일 수 있다.
```
gcloud compute instances create [만들 인스턴스 이름] --machine-type n1-standard-2 --zone $ZONE
```
그렇다. 위 명령어는 이전 포스팅에 있던 명령어와 매우 비슷하며, --zone의 Parameter로 위에서 설정한 환경 변수를 넣어준 것 뿐이다.  
위처럼, 환경 변수를 사용하면 스크립트를 조금 더 빠르고 편하게 작성할 수 있다.

## gcloud 명령어 더 둘러보기
> gcloud tool은 -h(help) flag를 제공한다. 따라서, 어떠한 gcloud command에도 맨 뒤에 -h를 붙이면 관련 설명을 볼 수 있다.

* help flag 사용 예시
```
gcloud -h
```
```
gcloud config --help
```
```
gcloud help config
```

* 사용자 환경의 구성 목록 조회
```
gcloud config list
```
```
gcloud config list --all
```
더 자세히 보려면 --all 플래그를 붙이면 된다.

* 컴포넌트 목록 조회
```
gcloud components list
```

## filtering을 활용한 gcloud 명령어 사용
> gcloud는 다양한 기능을 갖추고 있는데, 그 중 하나가 filtering 기능이다. 사용자가 원하는 대로 가공하여 정보를 볼 수 있다.

* Project 내 instances 목록 조회
```
gcloud compute instances list
```
위 명령어를 통해 우리는 프로젝트 내 존재하는 모든 인스턴스들의 목록을 아래와 같이 볼 수 있다.
```
NAME                   ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
cloudlearningservices  us-central1-a  e2-standard-2               10.128.0.2   35.202.228.143  RUNNING
gcelab2                us-central1-b  n1-standard-2               10.128.0.2   34.68.7.186     RUNNING
```

* NAME에 대한 filtering 적용
```
gcloud compute instances list --filter="name=('gcelab2')"
```
NAME 컬럼이 gcelab2인 결과만 보여주도록 설정한 것이다. 따라서 위 명령어의 결과는 아래와 같다.
```
NAME                   ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
gcelab2                us-central1-b  n1-standard-2               10.128.0.2   34.68.7.186     RUNNING
```

* Firewall rules 조회 명령어에 filtering 적용 예시
```
gcloud compute firewall-rules list
```
```
gcloud compute firewall-rules list --filter="network='default'"
```
```
gcloud compute firewall-rules list --filter="NETWORK:'default' AND ALLOW:'icmp'"
```
위 처럼 AND 연산자를 사용하여 이중 필터를 걸어줄 수도 있다. 참고로 여기서 NETWORK, ALLOW는 firewall-rules list의 컬럼이다.

## Firewall Rules 설정
> 우리가 만든 instances는 클라우드 상에 올라가기 때문에 방화벽과 같은 보안 규칙이 잘 되어있지 않으면 해킹의 우려가 있다.  
따라서 우리는 방화벽에 대한 이해와 활용이 매우 중요하다. 지금부터는 기본적인 방화벽 규칙 설정법에 대하여 배워보겠다.

* Firewall rules 조회
```
gcloud compute firewall-rules list
```
앞선 예시에서 자주 나왔던 gcelab2라는 이름의 Virtual machine에 Webserver인 nginx가 설치되어 있다고 가정해보자.  
nginx는 tcp:80포트를 통해 접근할 수 있는데, gcelabe2 Virtual machine의 external IP:80 포트로 접근해보면  
연결이 안된다는 것을 확인할 수 있다.  
그 이유는, 우리가 gcelab2에 대한 적절한 방화벽 규칙을 설정해주지 않아서인데, gcelab2가 tcp:80포트에 대한 연결을  
허용할 수 있도록 방화벽 규칙을 설정해보겠다.  

* 우선 큰 순서는 다음과 같다.  
1. gcelab2에 태그를 붙인다. (후에 방화벽 규칙의 Target을 tag로 지정하기 때문)
```
gcloud compute instances add-tags gcelab2 --tags http-server, https-server
```
2. tcp:80포트를 허용하는 방화벽 규칙을 만든다. (이때, 해당 방화벽 규칙의 target은 gcelab2의 태그로 설정)
```
gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```

위에서 만든 방화벽이 잘 생성되어 있는지 아래 명령어를 통해 확인해보자.
```
gcloud compute firewall-rules list --filter=ALLOW:'80'
```
방화벽 설정이 모두 끝났으니, 이제 최종적으로 gcelab2 인스턴스에 nginx server에 연결이 잘 되는지 아래 명령어를 통해 확인해보자.
```
curl http://$(gcloud compute instances list --filter=name:gcelab2 --format='value(EXTERNAL_IP)')
```

## References
> https://cloud.google.com/ 