---
title: GKE(Google Kubernetes Engine) Essentials
categories: [Computer engineering, Cloud infrastructure]
tags: [cloud infrastructure, google cloud platform, 인프라, 클라우드, 가상머신, 구글 클라우드 플랫폼]
---

이번 포스팅은 GKE(Google Kubernetes Engine)을 처음 사용해보는 사용자들을 위해 기초적이고 핵심적인 부분만 다뤄보려고 합니다.
GCP(Google cloud platform)에 대한 지식이 전혀 없다면 이전 포스팅인 [Google cloud CLI 포스팅](https://l-o-g-a-n.github.io/posts/gcloud-cli-essentials/)을 반드시 읽어본 뒤에 본 포스팅을 시작할 것을 추천드립니다.

## GKE(Google Kubernetes Engine)이란?
> GKE는 GCP 상에서 제공되는 구글 쿠버네티스 엔진으로 오랜 상업용 컨테이너 운영 경험의 노하우가 녹아있는 구글의 쿠버네티스 관련 서비스입니다. GKE는 구글 클라우드의 다양한 인스턴스 및 서비스 등을 기반으로 사용하며 이들이 주는 이점을 모두 누릴 수 있습니다. GCP가 주는 장점 몇가지를 살펴보면 다음과 같습니다.
1. [Load Balancing](https://cloud.google.com/compute/docs/load-balancing-and-autoscaling)
2. [Node Pools](https://cloud.google.com/kubernetes-engine/docs/concepts/node-pools)
3. [Automatic scaling](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler)
4. [Automatic upgrades](https://cloud.google.com/kubernetes-engine/docs/how-to/node-auto-upgrades)
5. [Node auto-repair](https://cloud.google.com/kubernetes-engine/docs/how-to/node-auto-repair)
6. [Logging and Monitoring](https://cloud.google.com/stackdriver/docs/solutions/gke)
이러한 서비스들을 바탕으로 GKE는 Kubernetes cluster를 구성하는데 도움을 주고 이들을 효율적으로 관리하기 위한 다양한 기능들을 제공합니다. 지금부터는 이러한 GKE를 통해 컨테이너 어플리케이션을 어떻게 배포할 수 있는지 알아보도록 하겠습니다.

## Default compute zone 설정
> 이전 포스팅을 통해 Zone과 Region의 개념에 대해서 익히 들으셨을 것이라고 생각합니다. 혹시나 이에 대한 이해가 부족하다면, [Google Cloud Platform Essentials 포스팅](https://l-o-g-a-n.github.io/posts/google-cloud-essentials)을 참고하시기 바랍니다. 우리는 앞으로 만들 우리의 Cluster와 그들의 resources가 위치할 디폴트 Zone을 아래와 같이 설정해줄 수 있습니다. 여기서는 us-central1-a Zone을 선택하였습니다.
```
gcloud config set compute/zone us-central1-a
```

## GKE cluster 생성
> Cluster는 최소 1개 이상의 cluster master machine과 nodes라고 불리는 다수의 worker machines으로 이루어집니다. Virtual machine instances가 이러한 nodes로 역할하게 됩니다. GCP에서는 Compute Engine virtual machine instances를 활용하여 워커 노드를 구성합니다. 클러스터를 생성하기 위해서는 아래 명령어를 사용하면 됩니다.
```
gcloud container clusters create [생성할 클러스터 이름]
```
수 분의 시간이 흐른 뒤, 생성이 완료되면 아래와 같은 출력 결과를 얻을 수 있을 것입니다.
```
NAME: my-cluster
LOCATION: us-central1-a
MASTER_VERSION: 1.21.5-gke.1302
MASTER_IP: 34.69.232.119
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.21.5-gke.1302
NUM_NODES: 3
STATUS: RUNNING
```

## Authentication credentials for cluters 얻기
> 클러스터를 생성한 뒤 해당 클러스터와 통신하려면 우리는 Authentication credentials, 즉 인증 자격 증명이 필요합니다. 우리가 접근하려는 클러스터의 이름을 사용하여 아래 명령어와 같이 자격 증명을 불러올 수 있습니다.
```
gcloud container clusters get-credentials [접근할 클러스터 이름]
```
위 명령어를 실행한 뒤 아래와 같은 결과가 나오면 성공입니다.
```
Fetching cluster endpoint and auth data.
kubeconfig entry generated for my-cluster.
```

## Containerized application 클러스터 배포
> 이제 클러스터에 본격적으로 application을 배포할 수 있습니다. 우리는 그중에서도 컨테이너화된 어플리케이션을 배포해보려고 합니다.
아래 예시에서는 Container Registry 내에 존재하는 hello-app이라는 컨테이너 이미지를 배포합니다.  
그 전에 앞서서, GKE에서는 클러스터의 자원들을 관리하고 생성하기 위해서 아래와 같은 Kubernetes Object를 사용합니다.
1. Deployment object: web server와 같은 stateless application을 배포하기 위해 사용
2. Service object: 배포할 application을 위한 규칙과 load banlancing을 정의하기 위해 사용  
우선, hello-app이라는 컨테이너 이미지로부터 hello-server라는 이름의 deployment object를 아래와 같이 생성해보겠습니다.
```
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```
아래와 같은 결과가 나왔다면 성공입니다.
```
deployment.apps/hello-server created
```
다음으로 외부 트래픽에 우리의 어플리케이션을 노출시키기 위해 Kubernetes service object를 생성해보겠습니다. 
```
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```
port flag를 통해 컨테이너가 노출될 포트를 지정해주었고, 컨테이너를 위한 Load Balancer를 생성하도록 type flag를 설정해주었습니다. 아래와 같은 결과가 나왔다면 성공입니다.
```
service/hello-server exposed
```
클러스터 배포가 모두 성공적으로 이루어졌는지 확인하기 위해 아래 명령어를 사용할 수 있습니다.
```
kubectl get service
```
위 명령어를 사용하면 아래와 같이 위에서 생성한 hello-server service가 생성되어 동작함을 확인할 수 있습니다.
```
NAME              TYPE              CLUSTER-IP        EXTERNAL-IP      PORT(S)           AGE
hello-server      loadBalancer      10.39.244.36      35.202.234.26    8080:31991/TCP    65s
kubernetes        ClusterIP         10.39.240.1                  433/TCP           5m13s
```
위 결과에서 나온 hello-server의 EXTERNAL-IP를 바탕으로 
```
http://[hello-server의 EXTERNAL-IP]:8080
``` 
브라우저를 통해 위 주소로 접속하게 되면 예제가 실행됨을 확인할 수 있습니다.

## GKE cluster 제거
> 위에서 생성한 클러스터를 제거하고 싶다면 아래와 같은 명령어 한 줄로 삭제할 수 있습니다.
```
gcloud container clusters delete [지우고자 하는 클러스터 이름]
``` 

## References
> https://cloud.google.com/  
https://www.cloudskillsboost.google/focuses/878?parent=catalog  
