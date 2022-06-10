---
title: Google Cloud Platform Essentials
tags: [Kubernetes, issue]
style: fill
color: success
description: GCP(Google Cloud Platform) 입문자를 위한 기본 지식
---

이전 포스팅들을 통해 알 수 있듯이, 현재 미국 Purdue 대학교에서 Kubernetes와 LoRa Network를 활용한 연구를 진행중이다.  
연구 실험을 위한 서비스들은 모두 클라우드 기반으로 구현하기로 했다.   
크고 다양한 클라우드 제공 업체들이 있지만, 그중에서도 우리는 GCP(Google Cloud Platform)을 사용하기로 했다.  

* 왜 GCP를 선택하였는가?
> 사실 처음엔 클라우드 서비스 업체 점유율 1위인 AWS를 아무 생각 없이 사용하려고 했다. 하지만, AWS 같은 경우엔 복합적인 이유로  
학교 측에서 비용 지원이 어렵다고 하여 차선택인 GCP를 사용하게 되었다. 근데, 막상 선택하고보니 잘 선택했다는 생각이 든다.  
우선 우리의 연구 핵심인 Kubernetes는 처음에 Google 팀에서 개발한 것이다. 그렇다보니 GKE(Google Kubernetes Engine)와 같은   
쿠버네티스 관련 서비스도 뭔가 더 강력할 것 같고, 사실 그보다도 AWS의 Kubernetes 서비스인 EKS(Elastic Kubernetes Service)보다  
GCP의 GKE가 더 직관적이고 관리하기 쉽게 UI/UX가 구성되어 있다는 점이 굉장히 큰 매력으로 다가왔다.

다 각설하고, 그래서 이제부터 GCP를 사용하기 위한 기본적인 개념과 구조들을 정리하고자 한다.

## GCP(Google Cloud Platform)이란?
Google Cloud Platform은 이름으로부터 쉽게 유추할 수 있듯이, Google 인프라 기반으로 호스팅되는 다양한 클라우드 서비스들의 모음이다.  
컴퓨팅, 스토리지, 데이터 분석, 머신러닝, 네트워킹 등 Google Cloud는 개인용부터 엔터프라이즈 급에 이르는 모든 클라우드 컴퓨팅 애플리케이션   
또는 프로젝트에 통합할 수 있는 다양한 서비스 및 API를 제공한다. 그 중에서도 우리는, GCP를 사용한다면 가장 먼저 접할 가능성이 높은   
가상 머신 서비스(Compute Engine)에 대해 알아보고자 한다.

## Compute Engine이란?
GCP 내 Compute Engine이라는 서비스가 있다. 이를 통해 우리는 Google 인프라 위에 다양한 OS를 실행시킬 수 있는 가상 머신과   
데이터를 저장할 스토리지 등을 만들 수 있다. 우리는 이러한 Resources를 필요한만큼 Hardware의 성능과 개수를 선택하여 구매할 수 있다.   
이에 따라 발생하는 비용은 사용한만큼 매달 청구된다. 이렇게 만들어진 Compute Engine resources는 Region 또는 Zone에 위치하게 된다.   
우리는 이러한 Region과 Zone에 대한 개념에 대해 명확히 구분지을 필요가 있다.

## Regions과 Zones이란?
앞서 알아본 것처럼 Compute Engine resources는 Region 또는 Zone에 위치하게 되는데, Region은 resources를 실행할 수 있는  
특정 지리적 위치이다. 그리고 이러한 Region은 1개 이상의 Zone을 포함한다.  
예를 들어, us-central1이라는 Region은 Central United states를 담당하며 이러한 Region은 us-central1-a, us-central1-b,  
us-central1-c, and us-central1-f 와 같은 총 4개의 Zone을 가지고 있다.  
Zone에 위치하는 resources를 Zonal resources라고 하며, Virtual machine instances와 Persistent disks들이 Zone에 위치한다.
만약 Persistent disks를 Virtual machine instances에 붙이고 싶다면 두 resources는 무조건 같은 Zone에 위치해야 한다.  
이와 유사하게, instance에 static IP를 부여하고 싶다면 instance와 static IP는 같은 Region에 있어야 한다.

## Virtual machine instances 생성 방법?
GCP를 통해 Virtual machine instances를 생성하는 방법에는 크게 2가지가 있다.  
첫번째는 GCP 웹 브라우저 콘솔을 이용해 GUI기반으로 생성하는 방법이고, 두번째는 GCP의 Cloud shell을 통해 명령어로 생성하는 방법이 있다.
GUI를 기반으로 생성하는 것은 크게 배경 지식이 없어도 가능하므로 생략하도록 하고 여기선 Cloud shell을 통한 instance 생성에 대하여 다룬다.
 
```
gcloud compute instances create [만들 인스턴스 이름] --machine-type n1-standard-2 --zone us-central1-f 
```
위 명령어처럼 Cloud shell에 입력해주면 바로 instance 생성이 가능하다. 이렇게 만들어진 instance에 SSH 연결도 가능한데,
```
gcloud compute ssh [연결할 인스턴스 이름] --zone us-central1-f
```
이렇게 입력해주면 위에서 만들었던 인스턴스에 바로 SSH 연결이 가능하다.  
다음 포스팅에서는 오늘 Cloud shell에서 사용한 gcloud command line tool에 대해 조금 더 깊이있게 다뤄볼 예정이다.

## References
> * https://cloud.google.com/