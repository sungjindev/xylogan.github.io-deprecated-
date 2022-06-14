---
title: 기술 면접_ 컴퓨터 메모리 구조
categories: [Computer science, Technical interview]
tags: [CS, 기술 면접, 메모리 구조, Computer science, Technical interview, Memory architecture]
---

**!본 포스팅은 기술 면접을 대비하기 위한 포스팅입니다.**
# 프로그램 실행 원리
![memory1](/assets/img/technical_interview/memory1.png){: w="50%" h="50%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

> 프로그램은 기본적으로 위와 같은 동작 순서로 실행됩니다.
1. 사용자가 운영체제에게 프로그램 실행 요청을 합니다.
2. 프로그램을 HDD나 SSD같은 보조기억장치로 부터 읽어 메모리에 로드합니다.
3. CPU는 메모리에 올라간 Code영역 내부에 프로그램 실행 명령어들을 통해 프로그램을 실행합니다.
4. 프로그램 내부 로직에서 동적으로 메모리를 할당하는 요청이 있으면 해당 부분은 Heap 영역에 할당합니다. 
5. 프로그램 내부 로직에서 함수가 호출되어 지역 변수 혹은 매개 변수를 사용하면 Stack 영역에 할당해줍니다.

# 메모리 공간 요소
![memory2](/assets/img/technical_interview/memory2.png){: w="50%" h="50%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

> 메모리는 위와 같이 크게 총 4가지의 영역으로 구분됩니다.
1. Code 영역
2. Data 영역
3. Heap 영역
4. Stack 영역

# Code 영역이란?
> **Code 영역**은 **프로그램의 소스 코드**가 담기는 부분입니다. 사용자가 프로그램을 실행하기 위해서 OS에 요청을 보내면 프로그램 실행 코드가 Code 영역에 담기게 되고, CPU가 Code 영역에 담긴 프로그램 실행 명령어들을 처리하여 실행하는 방식으로 프로그램이 실행됩니다.

# Data 영역이란?
> **Data 영역**은 **전역 변수와 Static 변수**가 할당되는 지역으로 전역 변수 및 Static 변수의 특징에 맞게 프로그램이 실행될 때 할당 되었다가 프로그램이 종료될 때 모두 할당 해제됩니다.

# Heap 영역이란?
> **Heap 영역**은 **프로그래머가 동적으로 할당하거나 해제할 때 할당받는 메모리 영역**입니다. 대표적으로 
C 언어에서 malloc()과 같은 메소드를 통해서 할당하고 free()라는 메소드를 통해서 할당 해제할 수 있습니다.
위 영역은 **런 타임**에 메모리 크기가 결정됩니다.

# Stack 영역이란?
> **Stack 영역**은 함수 호출 시 사용되는 **지역 변수 및 매개 변수**가 할당되는 영역입니다. 함수가 호출되면 생성되었다가 해당 함수가 종료되면 사라지는 **임시 메모리 공간**입니다.

# Heap과 Stack간의 관계
> **Heap과 Stack은 사실 하나의 메모리 공간을 나눠서 사용**합니다. 하나의 큰 메모리 공간을 Heap 영역은 메모리 주소상 아랫 주소부터 사용하고 Stack은 메모리 주소상 윗 주소부터 할당해 나가면서 서로 점유하는 형태입니다. 
그러다가 공유해서 쓰는 하나의 메모리 공간을 다 쓰게되면 서로 영역의 침범하게 되는데 그러한 상황을 **Heap overflow 혹은 Stack overflow**라고 부릅니다.

# References
> * https://jinshine.github.io/2018/05/17/%EC%BB%B4%ED%93%A8%ED%84%B0%20%EA%B8%B0%EC%B4%88/%EB%A9%94%EB%AA%A8%EB%A6%AC%EA%B5%AC%EC%A1%B0/
