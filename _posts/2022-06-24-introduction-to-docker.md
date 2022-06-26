---
title: Introduction to Docker
categories: [Computer engineering, Cloud infrastructure]
tags: [cloud infrastructure, google cloud platform, docker, 인프라, 클라우드, 구글 클라우드 플랫폼, 도커]
---

이번 포스팅에서는 GCP(Google Cloud Platform) 상에서 도커를 활용하는 방법에 대하여 배워보도록 하겠습니다. 도커에 대한 기본 개념이 부족한 분들에게도 도움이 될 것이라고 생각합니다. 

## Docker란?
> Docker는 컨테이너를 기반으로 하는 오픈소스 가상화 플랫폼입니다. 쉽게 말해 컨테이너 기반으로 어플리케이션을 작성하고 동작할 수 있게 해줍니다. 이렇게 만들어진 컨테이너는 시스템 환경에 구애받지 않고 도커 엔진만 있다면 어디서든 동일하게 실행시킬 수 있습니다. 이것이 컨테이너의 큰 장점입니다.

## Docker 실습 예제, Hello World
> 모든 프로그래밍의 기초 예제라고 할 수 있는 Hello World 예제를 통해 도커의 흐름을 알아보도록 하겠습니다. GCP Cloud Shell을 열고 아래의 명령어를 입력해봅니다.
```
docker run hello-world
```
그러면 아래와 같은 출력 결과가 나오는데, 이를 통해 다음을 알 수 있습니다.
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9db2ca6ccae0: Pull complete
Digest: sha256:4b8ff392a12ed9ea17784bd3c9a8b1fa3299cac44aca35a85c90c5e3c7afacdc
Status: Downloaded newer image for hello-world:latest
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```
docker daemon이 hello-world 이미지를 찾아봤지만 찾을 수 없었고, 그래서 Docker Hub라는 public registry에서 hello-world 예제 이미지를 pull 해오게 됩니다. 그렇게 받아온 이미지를 바탕으로 docker container를 만들게 되고 그 컨테이너를 실행하게 된 것입니다. docker image가 정상적으로 생성됐는지 아래 명령어를 통해 확인할 수 있습니다.
```
docker images
```
그리고 아래 명령어를 통해 docker container를 실행시킬 수 있습니다. hello-world 라는 이미지로부터 컨테이너를 실행시킨다는 의미입니다.
```
docker run hello-world
```
우리는 초기에 hello-world 실행시켰기 때문에 이미 pull 받은 이미지가 있어서 local 환경에서 해당 이미지를 바로 찾을 수 있습니다. 따라서 DockerHub에서 이미지를 불러오는 과정이 생략됩니다. 위 명령어를 통해 만들어진 이미지는 아래 명령어를 통해 현재 실행중인 컨테이너 목록들을 확인할 수 있습니다.
```
docker ps
```
하지만 정작 위 명령어를 실행해보면 실행되고 있는 컨테이너가 하나도 없음을 확인할 수 있습니다. 사실 hello-world 컨테이너는 이미 실행되고 끝났기 때문에 확인할 수 없는 것입니다. 이미 실행되고 끝나버린 컨테이너도 확인할 수 있는 명령어는 다음과 같습니다.
```
docker ps -a
```

## Docker build
> 테스트 환경을 구성하기 위해 우선 아래처럼 test 디렉토리를 만들고 해당 디렉토리로 이동해 줍니다.
```
mkdir test && cd test
```
그 후 Dockerfile을 아래와 같이 생성해줍니다.
```
cat > Dockerfile <<EOF
# Use an official Node runtime as the parent image
FROM node:lts
# Set the working directory in the container to /app
WORKDIR /app
# Copy the current directory contents into the container at /app
ADD . /app
# Make the container's port 80 available to the outside world
EXPOSE 80
# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF
```
각 line에 대한 설명은 주석으로 대체하도록 하겠습니다. 그 후 테스트용 node application을 실행해보기 위해서 app.js를 아래와 같이 만들어 줍니다.
```
cat > app.js <<EOF
const http = require('http');
const hostname = '0.0.0.0';
const port = 80;
const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello World\n');
});
server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});
process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
EOF
```
이제 미리 정의한 Dockerfile을 바탕으로 도커 이미지를 빌드할 차례입니다. 아래 명령어에 -t 플래그는 태그를 설정하는 플래그입니다. name:tag의 쌍으로 설정할 수 있습니다. 만약에 tag를 지정하지 않는다면 default로 latest라는 태그가 붙게 됩니다.
```
docker build -t node-app:0.1 .
```  
이렇게 빌드한 이미지는 아래 명령어를 통해 확인해볼 수 있습니다.
```
docker images
```
위 명령어를 실행한 결과는 아래와 같습니다.
```
REPOSITORY     TAG      IMAGE ID        CREATED            SIZE
node-app       0.1      f166cd2a9f10    25 seconds ago     656.2 MB
node           lts      5a767079e3df    15 hours ago       656.2 MB
hello-world    latest   1815c82652c0    6 days ago         1.84 kB
```
여기서 신기한 점을 확인해볼 수 있습니다. Dockerfile에서 FROM을 통해 지정한 node라는 이름의 base image까지 설치되어 있는 것을 확인해 볼 수 있습니다. 말 그대로 이건 base image이기 때문에 node-app 이미지를 지운 뒤에야 base image인 node image를 제거할 수 있습니다.

## Docker run
> 그럼 이제 위에서 build한 이미지를 기반으로 컨테이너를 run 시켜봅시다.
```
docker run -p 4000:80 --name my-app node-app:0.1
```
위 명령어를 통해 이미지를 바탕으로 컨테이너를 실행시킬 수 있습니다. 위에서 사용된 플래그에 대해서 자세히 알아보자면, name 플래그를 통해서 컨테이너 이름을 지정 할 수 있습니다. 또한 port 플래그는 호스트의 4000번 포트를 컨테이너를 외부에 노출 시킬 80포트와 매핑하는 것을 의미합니다. 컨테이너 실행이 정상적으로 되었는지 확인해보기 위해 curl 명령어를 통해 http request를 보내볼 수 있습니다. 하지만 위처럼 실행을 하면 명령어를 실행시킨 터미널이 켜져있을 때 까지만 컨테이너가 돌아가게 되고 터미널을 끄면 함께 종료됩니다. 이를 방지하기 위해 백그라운드에서 실행을 하고 싶다면 -d 플래그를 함께 적어주면 됩니다.
```
curl http://localhost:4000
``` 
이렇게 만든 컨테이너의 실행을 멈추고 제거하고 싶다면 아래 명령어를 사용하면 됩니다.
```
docker stop my-app && docker rm my-app
```

## Docker debug
> docker logs 명령어를 통해서 컨테이너의 로그를 확인할 수 있습니다.
```
docker logs -f [container_id]
```
실행중인 컨테이너에 bash로 접근하고 싶다면 docker exec 명령어를 사용하면 됩니다.
```
docker exec -it [container_id] bash
```
컨테이너의 metadata를 확인할 수 있는 명령어는 docker inspect입니다.
```
docker inspect [container_id]
```

## Docker publish
> 이미지를 Cloud에 배포하여 필요할 때 pull 받아 사용할 수 있습니다. 이를 위해 GCR(Google Container Registry)을 사용할 예정입니다. GCR에 이미지를 푸시하기 위해서는 먼저, [hostname]/[project-id]/[image]:[tag]와 같은 format으로 태그를 해줘야 합니다.
```
docker tag node-app:0.2 gcr.io/[project-id]/node-app:0.2
```
태그를 해준 뒤 아래 명령어를 통해 GCR에 push 해 줄 수 있습니다.
```
docker push gcr.io/[project-id]/node-app:0.2
```
GCR의 푸시된 이미지는 "Google cloud 상에 Container Registry"에서 확인할 수 있습니다. 이제부터는 GCR의 이미지를 pull 받아서 정상적으로 작동하는지 테스트를 해보겠습니다. 이를 위해 기존에 설치되어 있던 도커 이미지와 컨테이너를 모두 제거해주는 작업을 먼저 해주도록 하겠습니다. 우선 컨테이너의 실행을 멈추고 삭제하는 명령어입니다.
```
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
``` 
다음은, 순차적으로 도커 이미지를 제거해주는 명령어입니다.
```
docker rmi node-app:0.2 gcr.io/[project-id]/node-app:0.2 node-app:0.1
docker rmi node:lts
docker rmi $(docker images -aq) # remove remaining images
docker images
```
기존에 존재하던 도커 이미지와 컨테이너가 모두 제거되었으므로 GCR로부터 이미지 pull을 받고 테스팅 해보도록 합니다.
```
docker pull gcr.io/[project-id]/node-app:0.2
docker run -p 4000:80 -d gcr.io/[project-id]/node-app:0.2
curl http://localhost:4000
```

## References
> https://cloud.google.com/  
https://www.cloudskillsboost.google/focuses/1029?parent=catalog
