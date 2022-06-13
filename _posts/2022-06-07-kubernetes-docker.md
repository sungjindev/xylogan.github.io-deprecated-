---
title: Kubernetes의 Docker 지원 중단  
categories: [Computer engineering, Cloud infrastructure]
tags: [Cloud infrastructure, Kubernetes]
---

퍼듀대학교 K-SW Sqaure 프로그램에서, 쿠버네티스에 관한 주제를 가지고 분산 처리 컴퓨팅에 대한 연구를 진행하고 있습니다.  
이번 포스팅에서는 Kubernetes의 Docker 지원 중단 이슈에 대하여 살펴보고 어떻게 대처할 수 있는지 자세히 알아보려고 합니다.

## Kubernetes의 Docker 지원 중단 ? 
> Kubernetes에 관심이 있는 사람이라면, Docker 지원 중단에 대한 이야기를 한번쯤 들어봤을 것입니다.   
처음에 Docker 지원 중단에 대한 이야기를 들으면, 굉장히 당황할 수 있습니다. 하지만, 그 속사정을 들어보면 그들이  
왜 그러한 결정을 했는지 이해할 수 있습니다.  
Kubernetes는 다양한 기능들을 바탕으로 컨테이너들을 편하게 관리하는 오케스트레이션 툴입니다.     
컨테이너와 떼려야 뗄 수 없는 관계인 것입니다. 그러한 컨테이너들을 빌드하고 관리하는 패키지인, 도커에 대한 지원을 중단한다는 것이  
얼마나 당혹스러웠는지 본인도 잘 알고 있습니다.     
쿠버네티스는 컨테이너들의 이미지를 바탕으로 관리하는데, 도커 이미지는 OCI(Open Container Initiative)라는 표준을 따릅니다.   
이 표준에 부합한다면, 컨테이너를 도커로 빌드했던 안했건 도구에 관계없이 쿠버네티스 상에서는 똑같이 받아들입니다.   
그리고 쿠버네티스에서는 컨테이너 이미지를 가져오고 실행하는 역할을 하는 컨테이너 런타임이 필수적으로 필요한데, 
이를 위해 지금까지는 도커의 컨테이너 런타임,즉 Containerd를 사용하고 있었습니다.      
하지만, Docker는 CRI(Container Runtime Interface)라는 표준을 준수하지 않습니다. 이러한 표준을 맞춰주기 위해   
쿠버네티스는 Dockershim이라는 또 다른 도구를 활용하여 사용해왔습니다.    
이러한 과정에서의 의존성, 유지 보수성, 오버 헤드 등의 문제가 지속적으로 발생해왔습니다.   
그래서 Kubernetes 팀에서는 이러한 문제를 해결하고자 Container runtime으로 Docker를 거친 Containerd를  
사용하는 것이 아닌, Containerd와 같은 컨테이너 런타임 그 자체를 직접 사용하고자 한 것입니다.

## Docker 기반에서 Containerd로 마이그레이션
> 최근 쿠버네티스 릴리즈에서 실제로 도커에 대한 지원이 중단되면서 도커에서 제공하는 Containerd 런타임을 사용할 수 없게 되었습니다.  
따라서 우리는 CRI 표준을 준수하는 Containerd, CRI-O 등과 같은 컨테이너 런타임을 직접 설치하고 쿠버네티스가  
이를 인지할 수 있게 설정을 변경해주어야 합니다.  
컨테이너 런타임 설치는 다양한 경로와 방법이 공식 사이트에 잘 나와있으므로 생략하고, 여러 컨테이너 런타임 중에서 가장 흔히 쓰이는  
Containerd 설치가 완료된 상태에서 어떻게 쿠버네티스 설정을 변경할 수 있는지에 대해 적어보려고 합니다.

**/etc/containerd/config.toml** 경로에 있는 config.toml 파일에 들어가봅니다.
```
...
disabled_plugins = ["cri"]
...
```
Containerd를 RPM이나 .deb와 같은 패키지로부터 설치하게 되면, 디폴트 값으로 위와 같이 CRI integration plugin을 사용할 수 없게  
설정 되어있습니다. 위 문장을 주석처리하여 쿠버네티스가 Containerd를 사용할 수 있게 변경해줄 수 있습니다.   
> 파일 수정을 모두 마쳤다면,
```
sudo systemctl restart containerd
```
> 위 코드를 통해 업데이트 시켜주면 Kubernetes의 컨테이너 런타임 변경이 모두 완료됩니다.

## References
> https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/  
https://kubernetes.io/blog/2022/02/17/dockershim-faq/  
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd  
