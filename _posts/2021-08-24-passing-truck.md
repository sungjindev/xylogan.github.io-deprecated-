---
title: 고득점 Kit_ 스택/큐 - Level2 - 다리를 지나는 트럭
categories: [Computer science, Algorithm]
tags: [level2, programmers, stack, queue, 고득점, 다리를 지나는 트럭, 스택, 큐, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 고득점 Kit - [다리를 지나는 트럭](https://programmers.co.kr/learn/courses/30/lessons/42583) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <queue>
using namespace std;

int solution(int bridge_length, int weight, vector<int> truck_weights)
{
  int answer = 0;
  queue<int> entryTime;     //트럭이 완전히 올라갔을 때 진입 시간을 기준으로 저장
  queue<int> truckOnBridge; //올라가있는 트럭들
  for (int i = 1; i; i++)
  { //i가 여기서 timer 역할을 함
    if (i == entryTime.front() + bridge_length)
    {
      weight += truckOnBridge.front();
      truckOnBridge.pop();
      entryTime.pop();
    }
    if (weight - truck_weights[0] >= 0)
    {
      entryTime.push(i);
      weight -= truck_weights[0];
      truckOnBridge.push(truck_weights[0]);
      truck_weights.erase(truck_weights.begin());
      if (truck_weights.size() == 0)
        return answer = i + bridge_length;
    }
  }
}
```
지난번 위장 문제를 풀면서 크게 느꼈던 효율적인 알고리즘에 대한 고민을 이번엔 깊게 하면서 위 문제는 **어떻게 풀면 효율적인 로직이 나올 수 있을까를 가장 먼저 고민**하였습니다. 그 결과, 위 문제에서 **"트럭이 지나가는 순서가 정해져있으며 속도도 1초당 1로 모두 같으므로 트럭끼리 다리 위에서 역전될 일이 전혀 없으며 리턴 해야하는 결과값은 단지 시간 뿐이다."**라는 것을 알게 되었습니다. 따라서 저는 **FIFO 구조인 queue를 사용해야겠다**고 생각했고, 거기서 더 나아가서 **마지막으로 다리에 올라오는 트럭의 진입 시간을 알게되면 "해당 시간 + bridge_length"로 바로 정답을 구할 수 있음**을 깨달았습니다.

그래서 제 알고리즘을 보면, **진입 시간을 저장할 entryTime, 추후에 트럭이 다리에서 빠져나갈 때 얼마만큼의 무게를 가지는 트럭이 빠져나가는 지 알기 위해 올라가 있는 트럭들의 무게를 저장하는 truckOnBridge라는 2개의 queue를 활용하여 풀게됩니다.**
(**_추후 최적화한 알고리즘에서는 truckOnBridge라는 queue도 공간적 낭비가 있을 수 있음을 깨닫고 int형의 인덱스 포인터 2개로 대체하였습니다._**)

또 핵심적인 로직이라면 **for()문속 i라는 제어 변수를 마치 시간을 재는 timer 마냥 사용하면서 1초가 지나갈 때마다 다리 위에서 나가는 트럭은 없는지, 다리 위에 새로 진입하는 트럭은 없는지 검사**합니다. **그러다가도 마지막 트럭이 진입하면 곧바로 결과값을 리턴하면서 비효율적으로 돌만한 loop를 최소화** 시켰습니다.

## 다른 사람의 풀이1
``` cpp
#include <string>
#include <vector>
#include <queue>

using namespace std;

int solution(int bridge_length, int weight, vector<int> truck_weights)
{
  int answer = 0;

  int tot_w = 0;

  int t_front = 0;
  int t_cur = 0;

  int sec = 0;

  queue<int> q;

  while (t_front != truck_weights.size())
  {
    if (!q.empty() && (sec - q.front() == bridge_length))
    {
      tot_w -= truck_weights[t_front];
      ++t_front;
      q.pop();
    }
    if (t_cur != truck_weights.size() && tot_w + truck_weights[t_cur] <= weight)
    {
      tot_w += truck_weights[t_cur];
      ++t_cur;
      q.push(sec);
    }
    ++sec;
  }

  answer = sec;

  return answer;
}
```

위 풀이가 프로그래머스에서 가장 많은 추천을 받은 풀이입니다. 제 풀이의 로직과 굉장히 유사하지만,다른 점으로는 **truckOnBridge라는 queue대신 t_front, t_cur라는 index 포인터 역할을 하는 int형 type을 사용하면서 공간적 낭비를 최소화** 하였습니다. 또한 **위처럼 int 포인터를 사용하게 되면 제 알고리즘처럼 vector 내 erase 메소드를 사용할 필요도 없어지게 됩니다.** 

위와 반대로 저보다 **효율적이지 못한 부분**도 존재하는데요, 바로 **while()문에서 마지막 진입한 트럭을 찾았음에도 불구하고 sec를 계속해서 늘려가면서 마지막 트럭이 빠져나갈 때까지 반복문을 계속 돌린다는 점**입니다. 굳이 이럴 필요 없이 저처럼 **마지막 트럭을 찾게되면, "마지막 트럭의 진입 시간 + bridge_length"를 리턴해주면 비효율적인 loop를 최소화 시킬 수 있습니다.** 

## 다른 사람의 풀이2
``` cpp
#include <iostream>
#include <algorithm>
#include <functional> // greater 사용 위해 필요
#include <vector>
#include <queue>
using namespace std;

int solution(int bridge_length, int weight, vector<int> truck_weights)
{
  int answer = 0;
  int count = 0;
  int Time = 0;
  int Truck_weight = 0;
  queue<pair<int, int>> truck_move;

  while (true)
  {
    if (weight >= Truck_weight + truck_weights.at(count))
    {
      truck_move.push(make_pair(truck_weights.at(count), bridge_length + 1 + Time));
      Truck_weight += truck_weights.at(count);
      count++;
    }

    if (count >= truck_weights.size())
    {
      answer = truck_move.back().second;
      break;
    }
    else
    {
      Time++;
      if (truck_move.front().second == Time + 1)
      {
        Truck_weight -= truck_move.front().first;
        truck_move.pop();
      }
    }
  }

  return answer;
}
```

위 풀이도 대부분의 풀이와 유사하지만, **queue에 pair 타입을 사용**했다는 점이 이색적이어서 참고 삼아 가져와 봤습니다. 그 외의 로직에 대한 설명은 매우 비슷하기 때문에 생략하겠습니다.


## 최적화한 나의 풀이
``` cpp
#include <string>
#include <vector>
#include <queue>
using namespace std;

int solution(int bridge_length, int weight, vector<int> truck_weights)
{
  int answer = 0;
  int firstTruck = 0;   //다리 위 기준 가장 앞에 있는 트럭 인덱스
  int currentTruck = 0; //앞으로 다리 위로 가장 먼저 올라갈 트럭 인덱스
  queue<int> entryTime; //트럭이 완전히 올라갔을 때 진입 시간을 기준으로 저장
  for (int i = 1; i; i++)
  { //i가 여기서 timer 역할을 함
    if (i == entryTime.front() + bridge_length)
    {
      weight += truck_weights[firstTruck];
      entryTime.pop();
      firstTruck++;
    }
    if (weight - truck_weights[currentTruck] >= 0)
    {
      entryTime.push(i);
      weight -= truck_weights[currentTruck];
      currentTruck++;
      if (currentTruck == truck_weights.size())
        return answer = i + bridge_length;
    }
  }
}
```
앞서 언급됐던 알고리즘들의 장점이라고 생각한 부분들만 모아서 짜본 최적화 알고리즘입니다. **저처럼 queue를 2개 사용하는 것은 공간적인 측면에서 비효율적이라고 생각해서 queue 1개와 int type의 포인터 2개를 사용했고, for문은 기존의 제 알고리즘과 동일하게 작성**하였습니다.
