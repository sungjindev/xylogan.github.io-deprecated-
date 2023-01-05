---
title: 고득점 Kit_ 탐욕법 - Level1 - 체육복
categories: [Computer science, Algorithm]
tags: [level1, programmers, greedy, 고득점, 체육복, 탐욕법, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 고득점 Kit - [체육복](https://programmers.co.kr/learn/courses/30/lessons/42862) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <algorithm>
#include <iostream>

using namespace std;

int solution(int n, vector<int> lost, vector<int> reserve)
{
  int answer = 0;
  sort(lost.begin(), lost.end()); //lower_bound를 사용하기 위한 정렬
  for (int i = 0; i < reserve.size(); i++)
  { //여벌 체육복 있는 친구가 도난 당했을 경우에 대한 전처리
    vector<int>::iterator ptr1 = lower_bound(lost.begin(), lost.end(), reserve[i]);
    if (ptr1 != lost.end())
    { //lower_bound로 적절한 값을 찾지 못할 경우 .end()가 반환되므로 런타임 에러 발생 방지를 위하여 예외처리
      if (reserve[i] == *ptr1)
      {
        reserve.erase(reserve.begin() + i);
        lost.erase(ptr1);
        i--;
      }
    }
  }
  for (int i = 0; i < reserve.size(); i++)
  {
    vector<int>::iterator ptr2 = lower_bound(lost.begin(), lost.end(), (reserve[i] - 1));
    if (ptr2 != lost.end())
    {
      if ((reserve[i] - 1) == *ptr2)
      {
        reserve.erase(reserve.begin() + i);
        lost.erase(ptr2);
        i--;
        continue;
      }
    }
    vector<int>::iterator ptr3 = lower_bound(lost.begin(), lost.end(), (reserve[i] + 1));
    if (ptr3 != lost.end())
    {
      if ((reserve[i] + 1) == *ptr3)
      {
        reserve.erase(reserve.begin() + i);
        lost.erase(ptr3);
        i--;
      }
    }
  }
  answer = n - lost.size();
  return answer;
}
```
본 문제에도 잘 나와있지만, **여벌 체육복을 가지고 있지만 해당 학생이 도난을 당한 경우가 있을 수도 있습니다.** 따라서 저는 그 부분에 집중하여 가장 먼저 해당 이슈에 대한 전처리를 진행하였습니다.
위 전처리를 위해서 사용된 메소드가 lower_bound()입니다.

## lower_bound(), upper_bound()란?
> **lower_bound()는 C++ algorithm 라이브러리에 존재하는 메소드로 유사한 기능의 메소드로는 upper_bound()**가 있습니다. lower_bound()는 내부적으로 **이진 탐색**을 통해 **찾으려는 key값과 같거나 더 큰 값이 몇 번째 인덱스에서 처음 존재하는지 반환**해주는 메소드입니다.
내부적으로 이진 탐색을 활용하기 때문에 **탐색 대상이 되는 자료는 정렬이 되어있는 상태여야** 합니다.
``` cpp
template <class ForwardIt, class T>
ForwardIt lower_bound(ForwardIt first, ForwardIt last, const T& value);
ForwardIt upper_bound(ForwardIt first, ForwardIt last, const T& value);
```
lower_bound()와 upper_bound()는 위와 같은 원형을 가집니다.
이터레이터를 활용한 범위를 가지는 다른 대부분의 메소드와 마찬가지로 **[first, last)의 범위를 가진다는 점에 주의**하여야 합니다.
또한, **lower_bound()와 upper_bound()모두 원하는 키 값을 해당 범위에서 찾지 못할 경우에 last에 해당하는 iterator가 반환**됩니다.

이렇게 전처리를 해주고 난 뒤에 본격적으로 lost에서 reserve[i]의 전 후에 있는 학생들에게 체육복을 빌려줍니다. **우선적으로 reserve[i]의 앞에 있는 학생들의 체육복을 빌려주도록 설계**하였습니다. 왜냐하면, 예를들어 3번의 학생은 앞에있는 학생인 2번의 학생에게 빌려주고 6번 학생은 뒤에있는 7번 학생에게 빌려준다는 등 이런 식으로 **앞 뒤 혼용하여 빌려주게 되면, 최대한 많은 학생에게 빌려주기 위한 최적의 알고리즘이 나오지 않기 때문**입니다. 

## 다른 사람의 풀이1
``` cpp
#include <string>
#include <vector>

using namespace std;
int student[35];
int solution(int n, vector<int> lost, vector<int> reserve)
{
  int answer = 0;
  for (int i : reserve)
    student[i] += 1;
  for (int i : lost)
    student[i] += -1;
  for (int i = 1; i <= n; i++)
  {
    if (student[i] == -1)
    {
      if (student[i - 1] == 1)
        student[i - 1] = student[i] = 0;
      else if (student[i + 1] == 1)
        student[i] = student[i + 1] = 0;
    }
  }
  for (int i = 1; i <= n; i++)
    if (student[i] != -1)
      answer++;

  return answer;
}
```

프로그래머스에서 가장 많은 추천수를 받은 풀이 방법입니다. 제 풀이 방법과 많이 다릅니다. 저와 같은 경우에는 lower_bound()라는 메소드를 이번에 처음 알게되어서 공부하자는 의미로 대부분에 로직에서 lower_bound()를 사용했었습니다.
위 알고리즘의 큰 특징은 **넉넉한 크기의 student라는 전역 배열을 활용**하여 **해당 배열에 각 학생별로 가지고 있는 체육복의 개수를 카운팅**했다는 점입니다. 제 풀이방법과 관점이 조금 다릅니다.
여기서 가장 많이들 헷갈려하고 중요한 문법이 있습니다.

## for(int i: vector<int\> v)란? 
> 아래 코드에 집중해봅시다. 저도 이번 풀이 방법을 보면서 처음 알게된 문법입니다.
``` cpp
for (int i : reserve)
  student[i] += 1;
```
> 위 코드를 조금 더 이해하기 쉽게 바꾸면 아래와 같습니다.
``` cpp
for (int i=0; i<reserve.size(); i++)
  student[reserve[i]] += 1;
```
여기서 **주의해야할 것은 student 배열에 인덱스로 접근할 때 for문의 제어 변수인 i로 접근하는 것이 아니고 reserve[i] 값으로 접근**한다는 점입니다. 문제속 **reserve[i]에는 1보다 큰 값만이 들어있기 때문에 하단 부 for문의 제어변수 역시 1부터 순회**하도록 한 것입니다.

## 최적화한 나의 풀이
``` cpp
#include <vector>
#include <iostream>
using namespace std;

int student[35];
int solution(int n, vector<int> lost, vector<int> reserve)
{
  int answer = 0;
  for (int i = 0; i < reserve.size(); i++)
    student[reserve[i]]++;
  for (int i = 0; i < lost.size(); i++)
    student[lost[i]]--;
  for (int i = 1; i <= n; i++)
  {
    if (student[i] == -1)
    {
      if (student[i - 1] == 1)
        student[i - 1] = student[i] = 0;
      else if (student[i + 1] == 1)
        student[i] = student[i + 1] = 0;
    }
  }
  for (int i = 1; i <= n; i++)
    if (student[i] != -1)
      answer++;

  return answer;
}
```
최적화한 풀이는 "다른 사람의 풀이1"과 동일하여 자세한 설명은 이하 생략합니다.