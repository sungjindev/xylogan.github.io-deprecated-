---
title: 고득점 Kit_ 해시 - Level2 - 전화번호 목록
categories: [Computer science, Algorithm]
tags: [level2, programmers, hash, 고득점, 전화번호 목록, 해시, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 고득점 Kit - [전화번호 목록](https://programmers.co.kr/learn/courses/30/lessons/42577) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

bool solution(vector<string> phone_book)
{
  bool answer = true;
  sort(phone_book.begin(), phone_book.end()); //사전식 정렬을 통한 풀이
  for (int i = 0; i < phone_book.size() - 1; i++)
  {
    if (phone_book[i + 1].substr(0, phone_book[i].size()) == phone_book[i])
      return answer = false;
  }
  return answer;
}
```

우선 본격적인 설명을 하기 전에 앞서서 넋두리 좀 풀자면, 저는 이 문제 푸는데 **시간복잡도** 때문에 엄청나게 시간을 빼앗겼습니다... ~~_그러니 이 풀이를 보는 여러분들께서 많은 위안을 가져가셨으면 좋겠습니다._~~
제 알고리즘은 굉장히 심플합니다. ~~_그래서 오래 걸렸다는 것에 더 화가 납니다._~~
우선 정렬해줍니다. 여기서 제일 중요한 것이 있습니다. 너무 중요하니까 강조하고 넘어가겠습니다.

## sort()란?
> **sort()는 C++ algorithm 라이브러리 내에 있는 정렬 알고리즘**입니다. 앞선 포스팅에서도 설명한 적이 있지만 거듭 강조합니다. **sort(v.begin(), v.end(), compare);이런 식으로 사용할 수 있는데 compare 자리에 매개변수를 안넘겨주면 디폴트로 오름차순으로 정렬**됩니다.
_~~네... 이걸 설명하기 위해서 이렇게 강조를 한 것은 아니구요.~~_
이 문제에서 다시 한번 짚고 넘어가야 할 부분이 있습니다. **이 문제는 string vector**입니다.
다들 이미 알고 계셨겠지만 **string에서 오름차순이란 사전식 정렬**을 뜻합니다.
이게 중요합니다. _**사전식 정렬... 사전식 정렬... 사전식 정렬...**_
예를 들어서 **["123", "45", "456", "12", "333"] 이런 Vector가 사전식 정렬이 되면
["12", "123", "333", "45", "456"]이 되는 것**이죠. 

이제 감이 오시나요? 네... 사실상 문제 풀이 끝난 것입니다.
phone_book이 **사전식 정렬이 되었기 때문에 우리는 인접 원소끼리만 한번 비교**해주면 됩니다.
이렇게 time complexity를 줄인 것이죠.
**접두사로써 포함되는 지 관계 여부를 쉽게 따져주기 위해서** string에서 지원해주는 **substr()메소드를 활용**했습니다.

## substr()이란?
> **substr()은 string에서 지원해주는 함수**입니다. **부분 문자열을 리턴**합니다.
해당 함수의 원형은 아래와 같습니다.
``` cpp
basic_string substr(size_type pos = 0, size_type count = npos) const;
```
> 즉, **[pos, pos + count) 까지의 문자열을 반환**해주는 함수입니다.
우리는 접두사로 부분 문자열이 존재하는 지를 확인하면 되므로 
``` cpp
.substr(0, phone_book[i].size())
```
> 위처럼 활용할 수 있습니다.

## 다른 사람의 풀이1
``` cpp
#include <string>
#include <vector>
#include <algorithm>
#include <iostream>
using namespace std;

bool solution(vector<string> phoneBook) //나와 풀이가 동일하다.
{
  bool answer = true;

  sort(phoneBook.begin(), phoneBook.end());

  for (int i = 0; i < phoneBook.size() - 1; i++)
  {
    if (phoneBook[i] == phoneBook[i + 1].substr(0, phoneBook[i].size()))
    {
      answer = false;
      break;
    }
  }

  return answer;
}
```
해당 풀이가 프로그래머스 상에서 가장 많은 추천을 받은 풀이입니다. 저와 풀이가 동일해서 설명은 이하 생략하도록 하겠습니다.

## 다른 사람의 풀이2
``` cpp
#include <string>
#include <vector>
#include <unordered_map>

using namespace std;

bool solution(vector<string> phone_book) //unordered_map 해시를 이용한 풀이
{
  bool answer = true;

  unordered_map<string, int> hash_map;
  for (int i = 0; i < phone_book.size(); i++)
    hash_map[phone_book[i]] = 1;

  for (int i = 0; i < phone_book.size(); i++)
  {
    string phone_number = "";
    for (int j = 0; j < phone_book[i].size(); j++)
    {
      phone_number += phone_book[i][j];                            //한개의 전화번호를 한글자씩 떼어내서 phone_number에 기록
      if (hash_map[phone_number] && phone_number != phone_book[i]) //위에서 기록된 부분 전화번호인 phone_number가 hash_map에 있는지 확인!
        answer = false;
    }
  }

  return answer;
}
```

위 알고리즘은 Hash라는 문제 유형에 맞게 **C++에서 지원하는 해시인 unordered_map**을 활용한 풀이라고 할 수 있습니다. hash_map에 phone_book의 데이터들을 키값으로 넣어준 뒤에, **phone_number라는 빈 string에 phone_book[i]의 한글자씩 넣어가면서(부분 문자열인 셈) 해당 부분 문자열이 hash_map에 존재하는지 탐색**하는 방식입니다.
아래 **unordered_map 특징**에서도 나오지만, **탐색 시간복잡도가 O(1)**로 굉장히 빠르기 때문에 위 코드는 효율적으로 동작합니다.

## unordered_map이란?
> unordered_map은 C++에서 지원하는 해시 중 하나입니다. map과는 다소 다른 특징을 가지고 있습니다. **unordered_map의 특징**은 아래와 같습니다.<br>
**1. map보다 더 빠른 탐색을 하기 위한 자료구조 입니다.
2. unordered_map은 해쉬 테이블로 구현한 자료구조로 탐색 시간복잡도는 O(1)입니다.
3. map은 binary search tree로 구현되어 탐색 시간복잡도는 O(logn)입니다.
4. unordered_map은 중복 데이터를 허용하지 않으며 map에 비해 데이터가 많을 시 월등히 좋은 성능을 보여줍니다.
5. Key가 유사한 데이터가 많을 시 해시 충돌로 인해 성능이 떨어질 수 있습니다.**

## 최적화한 나의 풀이
최적화한 풀이가 처음에 본인이 푼 풀이와 동일하여서 이하 설명은 생략하도록 하겠습니다.

## References
> * https://math-coding.tistory.com/31