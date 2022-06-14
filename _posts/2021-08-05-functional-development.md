---
title: 고득점 Kit - 스택/큐 - Level2 - 기능개발
categories: [Algorithm, Programmers]
tags: [Level2, Programmers, queue, stack, 고득점, 기능개발, 스택, 알고리즘, 코딩테스트, 큐, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 고득점 Kit - [기능개발](https://programmers.co.kr/learn/courses/30/lessons/42586) 풀이입니다.**

# 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <iostream>
#include <vector>
#include <cmath>
#include <queue>
using namespace std;

vector<int> days;
vector<int> solution(vector<int> progresses, vector<int> speeds);
void calcDays(vector<int> progresses, vector<int> speeds);
vector<int> calcDistributes(vector<int> answer);

vector<int> solution(vector<int> progresses, vector<int> speeds)
{
  vector<int> answer;
  calcDays(progresses, speeds);
  answer = calcDistributes(answer);
  return answer;
}

void calcDays(vector<int> progresses, vector<int> speeds)
{
  for (int i = 0; i < progresses.size(); i++)
  {
    float number = ceil((100.0 - static_cast<float>(progresses[i])) / speeds[i]);
    days.push_back(number);
  }
}

vector<int> calcDistributes(vector<int> answer)
{
  int count = 0;
  int size = days.size();
  for (int i = 0; i < size; i++)
  {
    if (days[0] < days[1] || days.size() == 1)
    {
      answer.push_back(++count);
      count = 0;
      days.erase(days.begin());
    }
    else
    {
      count++;
      days.erase(days.begin() + 1);
    }
  }
  return answer;
}
```
우선, 각 기능별로 스피드에 대비해서 몇일의 시간이 소요되는지를 계산하고 저장하기 위한 days라는 전역 벡터를 두었습니다. 위 계산은 calcDays 메소드에서 이루어집니다. calcDays에서 나름 깔끔하게 짰다고 생각하는 코드가 아래 코드입니다. 효율적이라고 많은 추천을 받은 다른 풀이와 비교했을 때도 손색이 없는 코드라고 생각합니다. 올림을 통해 계산하였습니다.

``` cpp
float number = ceil((100.0 - static_cast<float>(progresses[i])) / speeds[i]);
```

그 후 calcDistributes 메소드에서 위에서 구한 days를 활용하여 각 배포시마다 몇개의 기능이 배포되는지 구할 수 있도록 구현하였습니다. 기본적으로는 days[0]과 days[1]을 비교하면서 count를 늘려주고 배포할 시점이 오면 answer 벡터에 push_back()해주는 방식으로 처리하였습니다.

# 다른 사람의 풀이1
``` cpp
vector<int> solution(vector<int> progresses, vector<int> speeds)
{
  vector<int> answer;

  int day;
  int max_day = 0;
  for (int i = 0; i < progresses.size(); ++i)
  {
    day = (99 - progresses[i]) / speeds[i] + 1; //내 코드 1줄과 같은 과정이지만 이개 더 깔끔하다. 99가 핵심이다.

    if (answer.empty() || max_day < day) //이 부분이 내 코드에 비해서 훨씬 효율적이다. 큰 영감을 얻을 수 있었다.
      answer.push_back(1);
    else
      ++answer.back();

    if (max_day < day)
      max_day = day;
  }

  return answer;
}
```
위 풀이는 프로그래머스에서 가장 많은 추천을 받은 풀이입니다. 저와 마찬가지로 작업 시간을 담는 day 변수가 있는데 제 코드보다 더 깔끔한 것 같습니다. 바로 아래의 코드입니다.

``` cpp
day = (99 - progresses[i]) / speeds[i] + 1;
```

사실 위 코드보다 아래의 알고리즘에서 더욱 큰 영감을 받을 수 있었습니다. max_day라는 변수를 통해 이전 탐색까지의 최대 작업 일수를 저장하고 for loop 속에서 max_day와 day를 비교하면서 배포 시점을 정해준 알고리즘입니다. 위 방식대로하면 처음에 짰던 제 코드보다 훨씬 더 짧고 이해하기 쉬운 코드를 구현할 수 있습니다.

# 다른 사람의 풀이2
``` cpp
vector<int> solution(vector<int> progresses, vector<int> speeds)
{
  vector<int> answer;

  queue<int> q; //queue를 활용한 풀이이다.

  for (int i = 0; i < progresses.size(); i++) 
  {
    int temp = (100 - progresses[i]) % speeds[i];
    if (temp == 0)
      q.push((100 - progresses[i]) / speeds[i]);
    else
      q.push((100 - progresses[i]) / speeds[i] + 1);
  }

  while (!q.empty()) //큐로 관리하는 부분이다.
  {
    int cnt = 1;
    int cur = q.front();
    q.pop();

    while (cur >= q.front() && !q.empty())
    {
      q.pop();
      cnt++;
    }

    answer.push_back(cnt);
  }
  return answer;
}
```

위 코드는 또다른 풀이법인데, 스택/큐 문제 유형에 잘 부합하게 큐를 통하여 풀이한 알고리즘입니다. 자료구조를 queue로 짰다는 것 외에는 거의 비슷한 로직이라서 이하 설명은 생략하도록 하겠습니다.

# 최적화한 나의 풀이
``` cpp
vector<int> days;
vector<int> solution(vector<int> progresses, vector<int> speeds);
void calcDays(vector<int> progresses, vector<int> speeds);
vector<int> calcDistributes(vector<int> answer);

vector<int> solution(vector<int> progresses, vector<int> speeds)
{
  vector<int> answer;
  calcDays(progresses, speeds);
  answer = calcDistributes(answer);
  return answer;
}

void calcDays(vector<int> progresses, vector<int> speeds)
{
  for (int i = 0; i < progresses.size(); i++)
  {
    float number = ceil((100.0 - static_cast<float>(progresses[i])) / speeds[i]);
    days.push_back(number);
  }
}

vector<int> calcDistributes(vector<int> answer)
{
  int maxDay = 0;
  for (int i = 0; i < days.size(); i++)
  {
    if (maxDay < days[i])
    {
      answer.push_back(1);
      maxDay = days[i];
    }
    else
      answer.back()++;
  }
  return answer;
}
```
제가 기존에 함수화 시켜놨던 모듈들은 그대로 유지시키고 싶어서, 다른 사람 풀이1에 비해서 비효율적이었던 calcDistributes() 메소드 부분을 위처럼 수정해주었습니다.  


