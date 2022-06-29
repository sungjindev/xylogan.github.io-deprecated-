---
title: 고득점 Kit_ 탐욕법 - Level2 - 큰 수 만들기
categories: [Computer science, Algorithm]
tags: [level2, programmers, greedy, 고득점, 큰 수 만들기, 탐욕법, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 고득점 Kit - [큰 수 만들기](https://programmers.co.kr/learn/courses/30/lessons/42883) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <algorithm>
#include <iostream>
using namespace std;

string solution(string number, int k)
{
  string answer = "";
  int size = number.size() - k;
  int i = 1;
  auto it = number.begin();
  while (answer.size() != size)
  {
    it = max_element(it, number.end() - (size - i));
    answer += *(it++);
    i++;
  }
  return answer;
}
```
위 문제를 풀 때 주의할 점은 parameter로 넘겨받는 **number에서 숫자를 제거만 할 뿐, 더 큰 숫자를 만들기 위해서 재배열을 할 수는 없다는 점**입니다. **즉 숫자들 사이에 이미 순서는 고정되어 있다는 점**

k는 제거할 숫자의 개수를 나타냅니다. numbers에 들어있는 총 숫자의 개수가 n이라고 해봅시다. 그러면, 제거할 거 다 제거한 뒤에 **최종 답인 answer의 size()는 n-k**가 될 것입니다.
**numbers 내부적으로 숫자들 간의 순서는 고정되어 있기 때문에 앞쪽에 있는 인덱스의 숫자가 최대값이 되어야 최종적인 answer도 최대값**이 될 수 있을것 입니다.즉, **numbers 내에서 Max값을 찾아줄겁니다.** 그런데 여기서 중요한 점이 있습니다.
**만약에 max값을 찾았는데 이 max가 뒤쪽 인덱스에 위치하고 있다면? 그래서 max 기준으로 뒤에 있는 모든 원소를 지우지 않고 살려 놓아도 answer.size()인 반드시 n-k보다 작게된다면? 그건 max로 쓸 수 없을 것입니다. 그래서 저는 0번째 인덱스부터 -n+k+i 미만의 원소 중에서 max_element()를 통해서 최대값을 찾아줄 것이고, 최대값의 이터레이터를 알게 되면 최대값 앞에 있는 모든 원소들은 제거 해줄 것 입니다. 근데 저는 제거해주는 시간조차 아깝다고 생각해서 실제로 원소를 제거하지는 않고 it의 위치만 바꿔주면서 마치 제거한 것처럼 처리하였습니다.**

## max_element()란?
> **C++ algorithm 라이브러리 내에 존재하는 최대값을 찾아주는, 정확히 말하면 최대값을 가지는 원소의 위치(이터레이터)를 리턴해주는 메소드**입니다. 이전 포스팅에서도 몇번씩 보셨을거라고 생각합니다.
보통은 **max_element(v.begin(), v.end());이런 식으로 벡터와 함께 많이쓰고, [v.begin(), v.end()) 범위 내에서 최대값을 찾아준답니다.**
**이번 알고리즘 문제에서는 벡터가 아니고 string 객체의 이터레이터를 활용해서 사용해주었습니다.
최소값을 구해주는 min_element()도 있으니까 참고하시면 좋을 것 같습니다.**

## 다른 사람의 풀이1
``` cpp
#include <string>
#include <vector>
#include <iostream>
using namespace std;

string solution(string number, int k)
{
  string answer = "";
  answer = number.substr(k);
  for (int i = k - 1; i >= 0; i--)
  {
    int j = 0;
    do
    {
      if (number[i] >= answer[j])
      {
        char temp = answer[j];
        answer[j] = number[i];
        number[i] = temp;
        j++;
      }
      else
      {
        break;
      }
    } while (1);
  }

  return answer;
}
```

위 풀이는 프로그래머스 상에서 가장 많은 추천을 받은 풀이입니다. 처음에 위 풀이를 봤을 때 너무 막막했습니다. 왜냐하면, for()문 안에 do while()문을 보는데 딱봐도 j가 지정된 인덱스를 벗어날 것만 같은 기분이 드는겁니다. 그래서, 위 알고리즘을 뜯어보기 전에 여러가지 테스트 케이스를 위 코드로 돌려봤습니다. 근데 완벽하게 정답.. 정답 ... 제 예상 혹은 걱정과 달리 위 코드는 아주 정상적으로 잘 작동했습니다. 오랜 시간 고민한 결과 그 이유를 알아냈습니다. 같이 한번 살펴봅시다.
우선 위 알고리즘에 대한 이해가 필요합니다.
``` cpp
answer = number.substr(k);
```
이 부분에서 answer에 임의의 부분 문자열을 넣어놓습니다. 여기서 중요한 것이 있는데 그 전에 substr()이 어떻게 동작하는 지 알아봅시다.

## substr()이란?
> **string 클래스에 내장되어 있는 메소드로 부분 문자열을 리턴해주는 함수입니다. 
정확히는 [pos, pos+count) 내의 부분 문자열을 리턴**해줍니다. 원형은 아래와 같이 생겼습니다.
``` cpp
basic_string substr(size_type pos = 0, size_type count = npos) const;
```
이렇게 원형만 놓고보면 이해도 잘 안되고 어려워보이니까 간단히 설명드리도록 하겠습니다.
보시면 알겠지만 **pos, count라는 parameter를 총 2개 받고 있고, pos, count의 Default값으로 각각 0과 npos가 설정**되어 있습니다. **count에 npos가 전달되면 자동으로 pos부터 문자열 끝까지 리턴**하게 되어있습니다. 아래 예제 케이스들을 보시면 더욱 이해가 잘 되실겁니다.

``` cpp
  // count가 npos 이므로 pos부터 문자열 끝까지 리턴한다.
  std::string sub1 = a.substr(10);
  
  // pos와 pos + count 모두 문자열 범위 안이므로, 해당하는 부분 문자열을 리턴한다.
  std::string sub2 = a.substr(5, 3);
  
  // pos는 문자열 범위 안이지만, pos+count는 밖이므로, pos부터 문자열 끝까지 리턴한다.
  std::string sub4 = a.substr(a.size() - 3, 50);
  
  // pos 가 문자열 범위 밖이므로 예외를 발생시킴.
  std::string sub5 = a.substr(a.size() + 3, 50);
```

자 다시 알고리즘 풀이로 돌아와서, 그러면 아까 해당 코드 부분을 다시 보겠습니다.
``` cpp
answer = number.substr(k);
```
그러면 이 말인즉슨, index k부터 마지막 원소까지를 부분 문자열로 리턴하겠다는 말일 것입니다.
이렇게 되면 **최종적으로 구해야하는 answer의 size()와 같은 길이를 가지는 string 객체가 answer에 들어가게 되는거입니다.** **이제 for()문을 들어가서 방금 만든 부분 문자열의 바로 왼쪽에 인덱스부터 시작하여 최대값을 찾아 탐색을 떠납니다.** **그러다가 do while()문에 들어가게 됩니다. 만약 왼쪽에 인덱스에서 더 큰 값이 발견되면 Swap()을 해주고 j를 1개 늘려줍니다. 이걸 늘리는 이유는, 방금 교체된 즉 answer안에 들어있던 그 원소가 다음 최대값이 될 수도 있으니까 answer내 나머지 원소들과도 모두 비교를 해주는 것 입니다.** **그런데 여기서 문제가 생깁니다. 이렇게 하다보면 분명히 100%, 아니 200% answer.size()를 넘어가는 out of range 문제가 생겨버립니다.** 제가 걱정했던 부분이 바로 이 부분입니다. **근데 결론부터 말씀드리면 문제가 없습니다.**
그 이유는 **저희가 다루는건 string 클래스의 객체입니다. 이렇게 대괄호를 써서 index로 접근하고 값 변경을 할 수도 있습니다. 하지만 index로 값을 넣어주고 한다고 해서 string 객체의 size()가 변하지는 않습니다.** 즉, 우리가 return하는 string answer의 size는 변하지 않는 것입니다. **제가 size를 넘어가는 범위에서 어떤 행동을 하건 그것은 제가 리턴하는 string answer에 영향을 미칠 수가 없습니다. cout으로 아무리 출력을 백번 천번 해봐도 answer.size()내의 결과값만 출력될 뿐입니다.**
**그렇다고 해서 string 내 오버라이딩 된 '+' 연산자도 똑같이 생각하시면 안됩니다.** **'+' 연산자는 두 문자열이 합쳐지고 하나의 새로운 크기의 문자열이 탄생하게 되어있습니다. 즉 해당 연산자를 사용하면 string의 size()도 알맞게 재조정이 됩니다.** 그렇게되면 우리가 원하는 answer.size()가 바뀌는 경우도 생길 것입니다.

## 다른 사람의 풀이2
``` cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

string solution(string number, int k)
{
  for (int i = 0; i < number.size() - k; ++i)
  {
    auto iter = std::max_element(number.begin() + i, number.begin() + i + k + 1);
    if (iter != number.begin() + i)
    {
      k = k - std::distance(number.begin() + i, iter);
      number.erase(number.begin() + i, iter);
    }
  }
  number.erase(number.end() - k, number.end());
  return number;
}
```
위 알고리즘도 프로그래머스 상에서 많은 추천을 받은 풀이인데, 제 알고리즘과 가장 비슷한 풀이입니다. 큰 알고리즘은 거의 동일하고 **하나 다르다면 이 알고리즘에서는 erase를 사용해서 진짜 원소들을 지워주면서 풀이**하고 있습니다. **그래서 max_element() 등과 같은 곳에서 접근하고 있는 이터레이터들이 조금씩 다릅니다.** 그 외에는 완전 동일해서 추가적인 풀이는 생략하도록 하겠습니다.

## 최적화한 나의 풀이
최적화한 풀이가 처음에 푼 풀이와 동일하므로 마찬가지로 생략하도록 하겠습니다.

## References
> https://m.blog.naver.com/kks227/220246803499  
https://modoocode.com/235