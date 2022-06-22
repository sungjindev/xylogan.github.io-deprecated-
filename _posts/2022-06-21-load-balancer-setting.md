---
title: Network and HTTP Load Balancers 구성
categories: [Computer engineering, Cloud infrastructure]
tags: [cloud infrastructure, google cloud platform, network load balancer, HTTP load balancer, 인프라, 클라우드, 가상머신, 구글 클라우드 플랫폼, 네트워크 로드 밸런서, HTTP 로드 밸런서]
---

이번 포스팅에서는 잠시 도커와 쿠버네티스를 내려놓고 GCP(Google Cloud Platform)에서 제공하는 Network load balancer와 HTTP load balancer에 대하여 알아보고 어떻게 구성할 수 있는지 실습과 함께 배워 보도록 하겠습니다.

## 로드 밸런서란?
> 로드 밸런서(load balancer)라는 이름에서 알 수 있듯이 부하를 분산시켜주는 역할을 합니다. 대규모 트래픽이 하나의 Instance에 집중적으로 들어오게 되면 우리는 당연스럽게도 병목 현상에 대하여 걱정할 것입니다. 이러한 문제들을 해결하고 서비스의 안정성과 신뢰성 등을 높이기 위해 load balancer를 사용합니다. GCP에서 제공해주는 다양한 로드 밸런서들이 있지만 그 중에서도 네트워크 계층 내 Layer 4 기반의 네트워크 로드 밸런서와 Layer 7 기반의 http 로드 밸런서에 대하여 공부해 보도록 하겠습니다.

## Default compute zone 및 region 설정
> 이전 포스팅을 계속해서 봐오셨던 분들은 이제 zone과 region의 개념에 대해서 충분히 익숙할 것이라고 생각합니다. 앞으로 실습을 통해 만들어질 instances들이 위치할 default zone과 region을 아래와 같이 설정해 줍니다.
```
gcloud config set compute/zone us-central1-a
gcloud config set compute/region us-central1
```   

## 다수의 web server instances 생성
> 로드 밸런싱의 효과를 테스트하기 위해 먼저 로드 밸런싱의 대상이 될 다수의 web server instances를 생성해주도록 하겠습니다. 총 3개의 compute engine virtual machines을 만들 것이며 그리고 해당 인스턴스에 Apache를 설치한 뒤, http traffic이 각각의 인스턴스들에 도달할 수 있도록 방화벽 규칙을 설정해 주도록 하겠습니다.  
우선, 아래 명령어를 통해 총 3개의 인스턴스를 만들어줄 것인데 추후에 방화벽 설정을 1개의 태그를 통해 쉽게 하기 위해 동일한 태그를 적용하여 묶어주겠습니다. 그 후 각 인스턴스가 시작될 때 다음과 같은 흐름을 가지는 스크립트를 실행하도록 하였습니다.  
"apache를 설치하고 테스트용 html 파일을 터미널에 출력한 뒤 해당 내용을 /var/www/html/index.html에 저장"
```
gcloud compute instances create www1 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www1</h1></body></html>' | tee /var/www/html/index.html"
```
```
gcloud compute instances create www2 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www2</h1></body></html>' | tee /var/www/html/index.html"
```
```
gcloud compute instances create www3 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www3</h1></body></html>' | tee /var/www/html/index.html"
```
그 후 생성된 각각의 인스턴스들에 http 트래픽이 닿을 수 있도록 아래와 같이 방화벽 규칙을 설정해 줍니다.
```
gcloud compute firewall-rules create www-firewall-network-lb \
  --target-tags network-lb-tag --allow tcp:80
```
정상적으로 설정되었다면, 아래 명령어를 통해 생성된 인스턴스들의 외부 IP를 확인하고 curl 명령을 통해 인스턴스들이 정상 동작중인지 확인해 봅니다.
```
gcloud compute instances list
curl http://[위에서 확인한 외부 IP]
```

## Network load balancer 구성
> 지금부터 본격적으로 네트워크 로드 밸런서를 구성해 볼텐데, 로드 밸런서를 구성할 때 각각의 가상 머신들은 자신이 설정한 정적 외부 IP 주소(static external IP address)를 통해 패킷들을 전달 받게 됩니다. Compute engine image로 만들어진 인스턴스들은 이 IP를 처리하도록 자동으로 구성됩니다. 우선 다음 명령어를 통해 정적 외부 IP 주소를 생성해 주도록 하겠습니다.
```
gcloud compute addresses create network-lb-ip-1 \
  --region us-central1
```
그 후 http health check resource과 동일한 지역의 인스턴스들의 묶음인 target-pool을 생성하고 이 pool에 health check를 적용해 줍니다.
```
gcloud compute http-health-checks create basic-check
gcloud compute target-pools create www-pool \
  --region us-central1 --http-health-check basic-check
```
위에서 생성한 pool에 아까 생성해두었던 3개의 인스턴스를 모두 추가해 줍니다.
```
gcloud compute target-pools add-instances www-pool \
  --instances www1, www2, www3
``` 
그 후 forwarding rules까지 추가해주면 이로써 Layer 4 네트워크 로드 밸런서 설정 완료입니다.
```
gcloud compute forwarding-rules create www-rule \
  --region us-central1 \
  --ports 80 \
  --address network-lb-ip-1 \
  --target-pool www-pool
```

## Network load balancer 테스트
> 위에서 구성한 Network load balancer을 테스트하기 위해 먼저 로드 밸런서에서 사용하는 forwarding rule의 외부 IP 주소를 확인해 줍니다.
```
gcloud compute forwarding-rules describe www-rule --region us-central1
```
외부 IP 주소를 확인했다면 curl 명령어를 통해 아래와 같이 위에서 생성한 3개의 인스턴스로부터 어떻게 response가 오는지 확인해 봅니다.
```
while true; do curl -m1 [위에서 확인한 외부 IP 주소]; done
```

## HTTP load balancer 구성
> 이번엔 Layer 4에 위치하는 네트워크 로드 밸런서가 아닌, Layer 7에 위치하는 HTTP 로드 밸런서를 구성해 보도록 하겠습니다. HTTP 로드 밸런서를 통해 URL 규칙 별로 보낼 인스턴스 설정할 수 있습니다. 또한, 사용자의 Request가 들어오면 항상 가장 가까우면서 충분한 Capacity를 가지고 있는 인스턴스로 라우팅 됩니다. Compute engine backend로 로드 밸런서를 구성하기 위해, Virtual machines들이 MIGs(managed instance groups) 안에 구성되어야 합니다. 이렇게 구성한 MIG은 외부 HTTP 로드 밸런서의 백엔드 서버를 실행시키는 가상 머신들을 제공합니다. MIG를 생성하기 전에 앞서서 아래 명령어를 통해 instance-templates를 만들어 줍니다.
```
gcloud compute instance-templates create lb-backend-template \
  --region=us-central1 \
  --network=default \
  --subnet=default \
  --tags=allow-health-check \
  --image-family=debian-9 \
  --image-project=debian-cloud \
  --metadata=startup-script='#! /bin/bash
    apt-get update
    apt-get install apache2 -y
    a2ensite default-ssl
    a2enmod ssl
    vm_hostname="$(curl -H "Metadata-Flavor:Google" \
    http://169.254.169.254/computeMetadata/v1/instance/name)"
    echo "Page served from: $vm_hostname" | \
    tee /var/www/html/index.html
    systemctl restart apache2'
```
이제 이 템플릿을 이용해 MIG를 만들어 줄 것입니다. MIG를 통해 동일한 여러 대의 VMs에서 어플리케이션을 동작시킬 수 있습니다. 또한, MIG는 autoscaling, autohealing, regional(multiple zone) deployment, and automatic updating 등의 자동화된 서비스를 제공합니다. 이러한 MIG 서비스를 이용하여 워크로드를 좀 더 가용성, 확장성 있게 만들어 줄 수 있습니다. 템플릿을 활용하여 MIG를 만드는 명령어는 아래와 같습니다.
```
gcloud compute instance-groups managed create lb-backend-group \
  --template=lb-backend-template --size=2 --zone=us-central1-a
```
그 후 추후에 Google Cloud health checking systems를 사용하기 위해 트래픽을 허용하는 방화벽 수신 규칙을 생성해 줍니다.
```
gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80
```
이제 HTTP 로드 밸런서를 위한 인스턴스들이 준비되었으므로, 사용자들이 접근할 수 있도록 정적 외부 IP 주소를 생성해 줍니다.
```
gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 \
  --global
```
위에서 설정한 정적 외부 IP 주소는 아래 명령어를 통해 확인할 수 있습니다.
```
gcloud compute addresses describe lb-ipv4-1 \
  --format="get(address)" \
  --global
```
로드 밸런서에 health-check를 적용하기 위해 아래처럼 만들어줍니다.
```
gcloud compute health-checks create http http-basic-check \
  --port 80
```
이제 로드 밸런서의 백엔드 서비스를 만들어준 뒤에 위에서 구성한 인스턴스 그룹을 해당 백엔드 서비스에 추가해줄 것입니다.
```
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global
```
```
gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=us-central1-a \
  --global
```
사용자가 보내오는 요청을 위에서 만든 backend service로 라우팅하기 위해 URL map을 만들어 줍니다.
```
gcloud compute url-maps create web-map-http \
  --default-service web-backend-service
```
이제 위에서 만든 URL map에 사용자 요청을 라우팅하기 위해 target HTTP 프록시를 만들어 줍니다.
```
gcloud compute target-http-proxies create http-lb-proxy \
  --url-map web-map-http
```
위 HTTP 프록시로 들어오는 사용자 요청에 대한 forwarding rule을 만들어주면 이로써 HTTP 로드 밸런서 구성이 끝이 납니다.
```
gcloud compute forwarding-rules create http-content-rule \
  --address=lb-ipv4-1\
  --global \
  --target-http-proxy=http-lb-proxy \
  --ports=80
```

## References
> https://cloud.google.com/  
https://www.cloudskillsboost.google/focuses/12007?locale=en&parent=catalog  
