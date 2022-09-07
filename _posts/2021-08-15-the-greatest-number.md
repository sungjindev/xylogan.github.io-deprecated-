---
title: 고득점 Kit_ 정렬 - Level2 - 가장 큰 수
categories: [Computer science, Algorithm]
tags: [level2, programmers, sort, 고득점, 가장 큰 수, 정렬, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 고득점 Kit - [가장 큰 수](https://programmers.co.kr/learn/courses/30/lessons/42746) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

bool compare(string lhs, string rhs)
{
  return lhs + rhs > rhs + lhs;
}

string solution(vector<int> numbers)
{
  string answer = "";
  int count = 0;
  vector<string> nums;
  for (int i : numbers)
    nums.push_back(to_string(i));
  sort(nums.begin(), nums.end(), compare);
  for (string i : nums)
  {
    answer += i;
    if (i == "0")
      count++;
  }
  if (count == nums.size())
    return answer = "0";
  return answer;
}
```

우선 위 풀이는 저 혼자만의 힘으로 푼 알고리즘이 아닙니다. 구조적인 부분은 굉장히 비슷하였지만 compare 함수 구현 중에 도저히 또다른 반례를 찾을 수 없어서 질문을 올리는 게시판에 들어가서 **반례를 찾다가 compare 구현 아이디어를 봐버렸습니다.**
저는 이중 If문 등을 활용하여 현재 compare()와 비슷한 로직을 가지는 구조를 구현하고 계속 시도하였지만 테스트 케이스 1~6번에서 계속 막혔었습니다. 그 중 위와같이 매우 간단한 문장으로 제가 생각했던 구조를 구현할 수 있어서 위 방법을 택했습니다.

그 외에 Solution 함수 부분에서 int를 모두 string으로 바꾸는 작업을 해주기 위해 string 라이브러리 내에 있는 to_string() 메소드를 활용하였습니다.

## to_string()이란?
> 메소드 명에서 유추할 수 있듯이 **string type으로 바꿔주는 함수**입니다. **C++ string 라이브러리 내에 존재**하며, 
``` cpp
to_string(123);
```
위와 같이 사용하면 됩니다.

## less<>, greater<>란?
> 오늘은 sort()와 아래와 같이 함께 쓰이는 less<>, greater<> 임시 객체에 대해 알아보겠습니다.
``` cpp
sort(v.begin(), v.end(), less<int>);
sort(v.begin(), v.end(), greater<int>);
```
위 이름에서 어느정도 유추할 수 있듯이, (왼쪽 매개변수 기준)less는 더 작은, greater는 더 크다는 의미를 가집니다. 즉 **less는 오름차순을 의미하고 greater는 내림차순**을 의미합니다.
**C++ 버전에 따라 less나 greater 뒤에 자료형 명시를 안해줘도 되는 경우도 있긴하나 낮은 버전에서는 타입 추론이 안되기 때문에 가급적이면 명시**를 해주는 것이 좋겠습니다.

## 다른 사람의 풀이
``` cpp
#include <algorithm>
#include <string>
#include <vector>

using namespace std;

bool compare(const string &a, const string &b)
{
  if (b + a < a + b)
    return true;
  return false;
}

string solution(vector<int> numbers)
{
  string answer = "";

  vector<string> strings;

  for (int i : numbers)
    strings.push_back(to_string(i));

  sort(strings.begin(), strings.end(), compare);

  for (auto iter = strings.begin(); iter < strings.end(); ++iter)
    answer += *iter;

  if (answer[0] == '0')
    answer = "0";

  return answer;
}
```
해당 풀이가 프로그래머스 상에서 가장 많은 추천을 받은 풀이입니다. 저와 풀이가 대부분 동일하며 하나 차이가 있다면, **[0, 0, 0] => "0" 과 같은 테스트 케이스**를 예외처리를 해주는 부분에서 저보다 간결하며 효율적입니다. 
저는 **제 알고리즘에서 Count라는 변수를 두어 처리를 해줬다면, 위 알고리즘에서는 이미 정렬되어있음을 그대로 이용하여 answer[0]을 통해 위 케이스를 걸러주었습니다.**
**compare 함수와 같은 경우에는 로직이 동일하지만 코드 가독성 측면에서 제 코드가 더 깔끔하다고 생각**합니다.

## 최적화한 나의 풀이
``` cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

bool compare(string lhs, string rhs)
{
  return lhs + rhs > rhs + lhs;
}

string solution(vector<int> numbers)
{
  string answer = "";
  vector<string> nums;
  for (int i : numbers)
    nums.push_back(to_string(i));
  sort(nums.begin(), nums.end(), compare);
  for (string i : nums)
  {
    answer += i;
  }
  if (answer[0] == '0') //이 부분이 최적화한 코드
    answer = "0";
  return answer;
}
```
많은 추천을 받은 효율적인 알고리즘이 제 로직과 크게 다르지 않다고 생각되어, 바로 위에서 언급했던 예외 케이스에 대한 부분만 조금 더 효율적으로 가다듬어 봤습니다.

## References
> https://blockdmask.tistory.com/334  
https://itguava.tistory.com/67