---
title: 고득점 Kit_ 깊이/너비 우선 탐색(DFS/BFS) - Level2 - 타겟 넘버
categories: [Computer science, Algorithm]
tags: [level2, programmers, BFS, DFS, 고득점, 타겟 넘버, 깊이/너비 우선 탐색, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 고득점 Kit - [타겟 넘버](https://programmers.co.kr/learn/courses/30/lessons/43165) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
using namespace std;

void PlusOrMinus(vector<int> numbers, int temp, int i, int target, int *answer)
{
  if (i == numbers.size())
    return;
  int temp1 = temp + numbers[i];
  int temp2 = temp - numbers[i];
  if (i == numbers.size() - 1 && temp1 == target)
    ++(*answer);
  if (i == numbers.size() - 1 && temp2 == target)
    ++(*answer);
  PlusOrMinus(numbers, temp1, i + 1, target, answer);
  PlusOrMinus(numbers, temp2, i + 1, target, answer);
}

int solution(vector<int> numbers, int target)
{
  int answer = 0;
  PlusOrMinus(numbers, numbers[0], 1, target, &answer);
  PlusOrMinus(numbers, -numbers[0], 1, target, &answer);
  return answer;
}
```

우선, 문제를 잘 이해해야 합니다. numbers라는 매개 변수로 숫자들이 넘어오게되는데 우리는 **이 각각의 숫자 앞에 + 혹은 -를 붙여서 계산**해야하고 **그 값이 target의 값과 일치하는 가짓수**를 찾는 문제입니다. 

저는 **PlusOrMinus라는 재귀 함수**를 두어서 이 문제를 풀었습니다. 
**이 재귀 함수 구조는 이진 트리**로 생각할 수 있는데요, **재귀함수가 호출되면 temp라는 노드의 값을 넘겨받고 그 temp 값에서 +와 -가지로 뻗어나가는 방식**입니다. **+쪽으로 뻗어나가면 temp1이라는 노드가 생기고, -쪽으로 뻗어나가면 temp2라는 노드가 생기도록 구현**한 것입니다.
그리고 PlusOrMinus라는 재귀 함수의 매개 변수 중에서 **int i를 활용하여 재귀 함수의 호출이 언제 중단되어야 하는지 종단점을 설정**해주었습니다.

## 다른 사람의 풀이1
``` cpp
#include <string>
#include <vector>

using namespace std;

int solution(vector<int> numbers, int target)
{
  int answer = 0;
  int size = 1 << numbers.size(); //비트 마스킹을 활용한 풀이이다. 다 적어보니 이해가 되었다.

  for (int i = 1; i < size; i++)
  {
    int temp = 0;
    for (int j = 0; j < numbers.size(); j++)
    {
      if (i & (1 << j))
      {
        temp -= numbers[j];
      }
      else
        temp += numbers[j];
    }
    if (temp == target)
      answer++;
  }
  return answer;
}
```
위 코드는 저에게 상당히 신성한 충격을 주었습니다. **비트 마스킹을 활용한 풀이법**입니다. 저는 비트를 가지고 놀기에는 수준이 부족하여 처음엔 코드 작동 원리를 이해하기 조차 버거웠습니다. 이 코드의 반복문을 따라 하나하나 이진수를 적어가니 그제서야 동작의 흐름이 보였습니다.
정말 간단히 설명하자면 **비트 마스킹 기법을 활용해서 제가 위에서 재귀 함수를 통해 구현했던 그런 이진 트리 탐색의 경우의 수를 모두 구현**해냈다고 생각하시면 됩니다. **하지만 위 풀이에 잘못된 점이 있습니다. 바깥 반복문에서 i=1로 초기화 되어있는데, i=0이 되어야 됩니다. i가 1부터 시작하게 되면 numbers가 1,1,1,1,1이고 target이 5일 때 정상적인 결과를 내지 못합니다.**

## 다른 사람의 풀이2
``` cpp
#include <string>
#include <vector>

using namespace std;

int total;

void DFS(vector<int> &numbers, int &target, int sum, int n) //나의 풀이와 매우 비슷하다.
{
  if (n >= numbers.size())
  {
    if (sum == target)
      total++;
    return;
  }

  DFS(numbers, target, sum + numbers[n], n + 1);
  DFS(numbers, target, sum - numbers[n], n + 1);
}

int solution(vector<int> numbers, int target)
{
  int answer = 0;

  DFS(numbers, target, numbers[0], 1);
  DFS(numbers, target, -numbers[0], 1);

  answer = total;

  return answer;
}
```
이 풀이는 제가 푼 풀이 방식과 굉장히 비슷하지만, 저보다 효율적인 코드입니다. 어떤 부분이 효율적인지 구체적으로 말씀을 드리자면 우선, **저같은 경우에는 +와 -가지로 나뉠 때 각각의 sum을 가지는 temp변수를 temp1, temp2로 총 2개를 나누어서 선언**했지만 **해당 코드는 int sum이라는 변수 하나로 통일**해서 가져가고 있습니다. **따라서 종단점을 설정해줄 때도 temp1, temp2에 대해서 나눠서 비교해줄 필요가 없으니 종단점을 설정해주는 if문의 길이도 짧아질 수 있었습니다.**

또 하나 다른 점이라면, **저는 재귀 함수를 호출하기 전에 따로 temp를 계산해주고 넣어주는 방식을 취했다면 해당 코드는 호출함과 동시에 temp를 계산해주었다는 점**이 차이점이라고 할 수 있을 것 같습니다.


## 최적화한 나의 풀이
``` cpp
#include <string>
#include <vector>
using namespace std;

void PlusOrMinus(vector<int> numbers, int temp, int i, int target, int *answer)
{
  if (i == numbers.size())
  {
    if (temp == target)
      ++(*answer);
    return;
  }
  PlusOrMinus(numbers, temp + numbers[i], i + 1, target, answer);
  PlusOrMinus(numbers, temp - numbers[i], i + 1, target, answer);
}

int solution(vector<int> numbers, int target)
{
  int answer = 0;
  PlusOrMinus(numbers, numbers[0], 1, target, &answer);
  PlusOrMinus(numbers, -numbers[0], 1, target, &answer);
  return answer;
}
```
그래서 저는 "다른 사람의 풀이2"에서 저보다 효율적이었던 부분을 참고하여 위처럼 수정해보았습니다. 위에서 자세히 과정을 설명해 놓았으니 이하 설명은 생략합니다.