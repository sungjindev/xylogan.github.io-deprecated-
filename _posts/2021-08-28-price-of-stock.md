---
title: 고득점 Kit_ 스택/큐 - Level2 - 주식가격
categories: [Computer science, Algorithm]
tags: [level2, programmers, stack, queue, 고득점, 주식가격, 스택, 큐, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 고득점 Kit - [주식가격](https://programmers.co.kr/learn/courses/30/lessons/42584) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <stack>
using namespace std;

vector<int> solution(vector<int> prices)
{
  vector<int> answer(prices.size(), -1);
  stack<int> s;
  stack<int> index;
  s.push(-1);
  for (int i = 0; i < prices.size(); i++)
  {
    if (s.top() > prices[i])
    {
      while (s.top() > prices[i])
      {
        int tempIndex = index.top();
        answer[tempIndex] = i - tempIndex;
        s.pop();
        index.pop();
      }
      s.push(prices[i]);
      index.push(i);
    }
    else
    {
      s.push(prices[i]);
      index.push(i);
    }
  }
  for (int i = 0; i < answer.size(); i++)
  {
    if (answer[i] == -1)
    {
      answer[i] = answer.size() - i - 1;
    }
  }
  return answer;
}
```
오늘은 스택/큐와 관련된 문제입니다. 출제 빈도가 높은 단골 유형입니다.
제 알고리즘을 보시면 아시겠지만, 저는 **스택 2개를 만들어서 문제를 풀이하였습니다.** 뒤이어 나오는 **다른 사람의 풀이1처럼 스택 1개만 가지고 훨씬 간결하고 직관적으로 풀이 할 수 있습니다.**
풀이 자체는 되게 간단합니다. **prices 배열의 원소인 주식 가격을 순서대로 차곡차곡 담아줄 s와, 해당 주식가격과 매핑되는 인덱스도 순서대로 차곡차곡 담아줄 index를 선언**해줬고 둘 다 자료구조는 stack을 사용했습니다. LIFO 구조가 필요했기 때문입니다. 
**주식 가격과 해당 가격에 매핑되는 인덱스로 pair로 매핑해줄까 하였지만, 급히 풀어서 그런지 그냥 index만으로 접근할 생각을 미처 못했습니다.**
네, 그 다음에 핵심 로직은 **스택 탑에 있는 주식 가격과 현재 들어와야할 주식 가격을 비교해서 들어올 주식 가격이 더 크면(이 시점이 주식 가격이 떨어지는 시점일 것입니다.) 그때부터는 주식 가격이 더 크거나 같은 값이 나올 때까지 stack pop() 해주면 됩니다.**
**제 알고리즘에선 index 관련된 정보는 index라는 stack에서 따로 저장하고 관리하고 있기 때문에 stack s에서 pop(), push()가 일어나면 index stack에서도 모두 동일하게 처리** 해줘야 합니다.

위 풀이에서 **맨 마지막 for 루프에 -1** 부분에 관하여 설명드리자면, **answer를 시작하기 전부터 모두 -1로 초기화 시켜놓고 answer에 변화가 생기면(-1이 아니면) 알고리즘을 거쳐서 적절한 답들로 변경되었다고 처리를 해준겁니다. 위 루프를 거쳐도 -1이라면, 끝까지 가격이 떨어지지 않고 살아남은 친구들이기 때문에 후처리를 해준 것입니다.**

다음 풀이로 넘어가기 전에 stack 자료 구조에 대해서 짚어봅시다.

## stack이란?
> **LIFO(Last In First Out) 구조로 이루어져 있으며 C++에서는 stack이라는 헤더파일 내에 들어 있습니다.** 
**stack.top()이라는 메소드를 통해서 top에 있는 원소를 참조할 수 있고 stack.pop()을 통해서 최상단의 원소를 지워낼 수 있습니다. stack.pop()을하면 리턴값으로 pop()처럼 stack top에 있는 원소가 리턴된다고 생각하기 쉽지만 그렇지 않으므로 주의하시길 바랍니다.**
그 이외에도 **스택에 원소를 넣어주는 stack.push()
스택이 비어있는지 확인해 주는 stack.empty()
스택 사이즈를 리턴해주는 stack.size()** 등이 있습니다.


## 다른 사람의 풀이1
``` cpp
#include <string>
#include <vector>
#include <stack>

using namespace std;

vector<int> solution(vector<int> prices)
{
  vector<int> answer(prices.size());
  stack<int> s;
  int size = prices.size();
  for (int i = 0; i < size; i++)
  {
    while (!s.empty() && prices[s.top()] > prices[i])
    {
      answer[s.top()] = i - s.top();
      s.pop();
    }
    s.push(i);
  }
  while (!s.empty())
  {
    answer[s.top()] = size - s.top() - 1;
    s.pop();
  }
  return answer;
}
```
위 알고리즘이 프로그래머스 내에서 가장 많은 추천을 받은 풀이입니다. 제 로직과 굉장히 비슷하고 stack을 활용한 것도 똑같습니다. 하지만 저보다 훨씬 직관적이고 효율적입니다. 왜냐하면 저는 index를 따로 관리하려고 stack을 하나 더 만들어서 총 2개의 stack을 사용했기 때문입니다.
근데 **위 알고리즘처럼 인덱스를 stack에 저장하면 index를 통해 prices에 접근해서 주식 가격도 얻을 수 있고, index도 그대로 활용할 수 있어서 훨씬 효율적입니다.**
그리고 **제 알고리즘에서 if-else 문으로 pop()+push(), push()를 나누어서 처리를 해줬다면 위 알고리즘에서는 깔끔하게 1개의 while()문과 push()로 잘 짜주었습니다.**
그 외의 로직은 상당히 비슷하기 때문에 생략하도록 하겠습니다.


## 다른 사람의 풀이2
``` cpp
#include <string>
#include <vector>
using namespace std;

vector<int> solution(vector<int> prices)
{
  vector<int> answer;
  int size = prices.size();

  for (int i = 0; i < size; i++)
  {
    int time = 0;
    for (int j = i + 1; j < size; j++)
    {
      time++;
      if (prices[j] < prices[i] || j == size - 1)
      { //마지막에서 두번째 값은 만약 마지막 값이 더 크다면 push할 수가 없기때문에 조건에 추가
        answer.push_back(time);
        break;
      }
    }
  }

  answer.push_back(0); //마지막은 시간이 없음

  return answer;
}
```
위 알고리즘은 이중 포문을 활용한 풀이입니다. 크게 이 문제는 이중 포문 혹은 stack을 활용한 풀이 이렇게 2가지로 나뉩니다. 이 알고리즘은 더 이해하기가 쉽습니다. **앞에 원소부터 차례차례 이중 포문을 돌면서 time을 늘려가면서 문제 조건에 부합하는 원소를 찾고 해당 원소가 나오면 answer에 바로 푸시합니다. 근데 포문들을 끝까지 돌았는데 조건에 부합하는 원소들이 있을 수 있습니다. 해당 경우를 대비해 if문에 j == size-1로 처리**해준 것을 볼 수 있습니다.

## 최적화한 나의 풀이
최적화한 풀이가 다른 사람의 풀이와 동일하여 생략하도록 하겠습니다.

## References
> https://twpower.github.io/75-how-to-use-stack-in-cpp
