---
title: 고득점 Kit_ 정렬 - Level1 - K번째수
categories: [Computer science, Algorithm]
tags: [level1, programmers, sort, 고득점, K번째수, 정렬, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 고득점 Kit - [K번째수](https://programmers.co.kr/learn/courses/30/lessons/42748) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <algorithm>
#include <iostream>
using namespace std;

vector<int> solution(vector<int> array, vector<vector<int>> commands)
{
  vector<int> originalArray = array;
  vector<int> answer;
  for (int i = 0; i < commands.size(); i++)
  {
    sort(array.begin() + commands[i][0] - 1, array.begin() + commands[i][1]);
    answer.push_back(array[commands[i][0] + commands[i][2] - 2]);
    array = originalArray;
  }
  return answer;
}
```
이 문제는 정렬 문제답게 정렬이 핵심이고 Vector로 인자가 넘어오기 때문에 Vector에 인덱스 접근을 잘하는 것이 매우 중요합니다. 저는 오름차순 정렬을 위해 algorithm 라이브러리에 있는 sort를 활용하였습니다.
## sort()란?
> **c++ algorithm 라이브러리**에 존재하는 정렬 알고리즘으로, 내부적으로 **퀵 정렬로 구현되어 있어서 시간 복잡도가 O(nlogn)으로 빠른 편**입니다. 함수의 원형은 아래와 같이 생겼습니다.
``` cpp
template <typename T>
void sort(T start, T end, Compare comp);
```
> 위에서 보면 알 수 있겠지만, 첫번째와 두번째 인자로는 정렬할 시작점과 끝점을 정해주게 됩니다.
여기서 가장 중요한 것은 정렬이 되는 범위입니다. 보통 [start, end]라고 오해하는 경우가 있는데
(~~*필자 본인 이야기*~~) 사실 정렬되는 범위는 **[start, end) 이므로 반드시 주의**해야 합니다.

그 외에 알고리즘은 **Vector의 이터레이터와 인덱스 접근**에 대하여 실수하지 않고 잘 이해하고 있다면 쉽게 이해할 수 있을 것입니다. 하나 특이한 점으로는 저는 originalArray라는 변수를 두어서 **기존에 넘어온 array 배열 자체에 조작**을 가하고, **for문의 1 cycle이 끝나면 다시 originalArray로 복원**해주는 방식을 선택하였습니다.


## 다른 사람의 풀이1
``` cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

vector<int> solution(vector<int> array, vector<vector<int>> commands)
{
  vector<int> answer;
  vector<int> temp;

  for (int i = 0; i < commands.size(); i++)
  {
    temp = array;
    sort(temp.begin() + commands[i][0] - 1, temp.begin() + commands[i][1]);
    answer.push_back(temp[commands[i][0] + commands[i][2] - 2]);
  }

  return answer;
}
```

프로그래머스에서 가장 많은 추천수를 받은 풀이 방법입니다. 제 풀이와 완전 동일하여서 스스로 놀랐고, 이하 설명은 생략하도록 하겠습니다.


## 최적화한 나의 풀이
최적화된 풀이가 필자가 처음에 푼 풀이와 동일하여 생략합니다.