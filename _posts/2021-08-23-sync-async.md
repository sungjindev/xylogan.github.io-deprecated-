---
title: 기술 면접_ 동기와 비동기
categories: [Computer science, Technical interview]
tags: [CS, 기술 면접, 동기, 비동기, computer science, technical interview, synchronous, asynchronous]
---

!본 포스팅은 기술 면접을 대비하기 위한 포스팅입니다.

## 동기식(Synchronous) 처리란?
> **동기식 처리란 직렬적으로 이루어지는 처리 과정**을 말합니다. **직렬적이란 작업이 순차적으로 처리되며 이전 작업이 끝나지 않으면 다음 작업이 시작되지 않음을 의미**합니다. 즉 아래 그림에서 Task1과 Task1 사이에 Get Data from Server 작업이 실행중이라면 Server로 부터 Data를 모두 받고나서야 뒤 Task1이 실행된다는 것을 의미합니다.  

![synchronous](/assets/img/technical_interview/synchronous.png){: w="80%" h="80%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

아래와 같은 코드가 동기식 처리의 예시가 될 수 있습니다.
``` js
function func1() { 
  console.log('func1');
  func2(); 
} 
function func2() { 
  console.log('func2');
  func3();
} 
function func3() { 
  console.log('func3');
}
func1();
```

## 비동기식(Asynchronous) 처리란?
> **비동기식 처리란 동기식 처리와 비교되는 개념으로 병렬적으로 이루어지는 처리 과정을 의미합니다. 병렬적이란 의미 답게 여러개의 작업이 동시에 처리될 수 있음을 뜻합니다.** 일반적으로 작업 마다 처리에 소요되는 시간이 다를 것이기 때문에 동시에 시작되더라도 먼저 끝날 수도 있고 늦게 끝날 수도 있습니다. 심지어 먼저 시작되더라도 늦게 끝날 수도 있는 것입니다. 아래 코드를 통해 이해해 봅시다.
``` js
function func1() { 
  console.log('func1');
  func2();
} 
function func2() { 
  setTimeout(function() { 
    console.log('func2');
  }, 0); 
  func3();
} 
function func3() { 
  console.log('func3'); 
} 
func1();
```
setTimeout()이 대표적인 비동기 처리 함수인데, 타이머를 0으로 설정하였음에도 불구하고 func1, func2, func3의 순서로 console에 로그가 찍히지 않습니다.  

![asynchronous](/assets/img/technical_interview/asynchronous.png){: w="80%" h="80%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}  

왜냐하면 **setTimeout()의 콜백함수는 비동기적으로 처리될 뿐만아니라 지정된 대기 시간만큼을 기다리다가 "tick" 이벤트가 발생하면 이벤트 큐로 이동하고, Call Stack이 모두 비어진 뒤에야 Call stack으로 이동하여 실행되기 때문**입니다. 위 과정을 그려보면 아래와 같습니다.  

![setTimeout](/assets/img/technical_interview/setTimeout.gif){: w="80%" h="80%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

## References
> https://webclub.tistory.com/605
