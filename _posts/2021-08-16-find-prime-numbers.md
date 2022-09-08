---
title: 고득점 Kit_ 완전탐색 - Level2 - 소수 찾기
categories: [Computer science, Algorithm]
tags: [level2, programmers, exhaustive search, 고득점, 소수 찾기, 완전탐색, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 고득점 Kit - [소수 찾기](https://programmers.co.kr/learn/courses/30/lessons/42839) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <set>
#include <algorithm>
using namespace std;
set<string> result;

void recursive(vector<string> temp, string i, string num)
{
  if (i != "0" || num.length() != 0)
  {
    num += i;
    result.insert(num);
  }
  temp.erase(find(temp.begin(), temp.end(), i));
  for (string i : temp)
    recursive(temp, i, num);
}

bool isPrimeNumber(string num)
{
  int number = stoi(num);
  if (number <= 1)
    return false;
  for (int i = 2; i < number; i++)
  {
    if (number % i == 0)
      return false;
  }
  return true;
}

int solution(string numbers)
{
  int answer = 0;
  vector<string> nums;
  for (char i : numbers)
  {
    nums.push_back(to_string(i - 48));
  }
  for (string i : nums)
    recursive(nums, i, "");
  for (string i : result)
  {
    if (isPrimeNumber(i))
      answer++;
  }
  return answer;
}
```

문제에 주어진 조건에 따르면 numbers는 길이가 7이하인 문자열이기 때문에 최대 7자리의 정수가 나오게 됩니다. 이 정수로 만들어질 수 있는 모든 숫자들을 구하는 로직이 관건이 될텐데 **처음에 저는 중첩 for문을 통해서 해결하려고 했으나 상당한 깊이의 중첩구조가 나오게 된다는 것을 깨닫고 재귀 함수를 이용하여 구현**하도록 하였습니다.

그래서 제 알고리즘의 핵심인 recursive() 재귀함수 먼저 살펴보도록 하겠습니다.
``` cpp
  if (i != "0" || num.length() != 0)
  {
    num += i;
    result.insert(num);
  }
```
우선 위 코드와 같은 경우에는 **맨 앞에 0이 오는 경우를 걸러주는 것**입니다. 즉, 0으로 시작하는 숫자가 만들어질 수 없도록 구현하였습니다. 미리 말씀드리자면 numbers 속 숫자들을 통해 만들어질 수 있는 숫자는 result라는 set 자료구조에 저장되도록 구현하였습니다. **set을 사용했기 때문에 중복된 key값은 알아서 제거되며, 디폴트로 오름차순으로 정렬**됩니다. set에 대해서 간단히 알아볼까요?

## set이란?
> **set은 set, multiset, map, multimap을 포함하는 연관 컨테이너(associate container) 중 하나**입니다. **연관 컨테이너란 key와 value처럼 관련있는 데이터를 하나의 쌍으로 저장하는 컨테이너**입니다. 그 중에서도 **set은 가장 단순한 연관 컨테이너**로, **value를 key값 그 자체로 사용**합니다. 또한 **set에서는 key 값의 중복을 허용하지 않고, multiset은 key 값의 중복을 허용**한다는 점에서 차이점을 가지고 있습니다.
``` cpp
set<int> s1;
```
위와 같이 선언하여 사용하며, **set에서는 기본적으로 오름차순에 맞게 정렬하여 데이터를 삽입**합니다.

다시 돌아와서, recursive() 함수에 대해서 조금 더 살펴보겠습니다. **recursive()에는 총 3개의 매개변수를 넘겨받습니다. temp에는 재귀 함수를 호출할 때마다 사용할 수 있는 numbers 내의 숫자들을 vector로 담고 있고, i에는 이번에 사용할 number를 지칭합니다. num에는 지금까지 numbers 내의 일부 숫자들을 이용해 구현된 부분 문자열을 담고 있습니다.**

따라서, i를 통해 이번에 사용할 number를 넘겨받고 i를 추가한 num을 result에 넣어줍니다. 그 후 사용한 i는 temp에서 제거해주는 로직입니다.

solution내에 사용한 로직을 설명하자면, **nums라는 string vector를 두었고 numbers 내의 각각의 문자를 아스키 코드와 to_string을 활용해서 하나하나의 string으로 변환하여 nums vector 내부에 삽입하였고 이 배열을 매개로 재귀 함수를 호출하여 문제를 해결**하였습니다. 

## 다른 사람의 풀이
``` cpp
#include <string>
#include <vector>
#include <iostream>
#include <algorithm>
#include <set>
#define MAX 9999999999
using namespace std;

bool isPrime(int number)
{
  if (number == 1)
    return false;
  if (number == 2)
    return true;
  if (number % 2 == 0)
    return false;

  bool isPrime = true;
  for (int i = 2; i <= sqrt(number); i++)
  {
    if (number % i == 0)
      return false;
  }

  return isPrime;
}

bool compare(char a, char b)
{
  return a > b;
}

int solution(string numbers)
{
  int answer = 0;

  string temp;
  temp = numbers;

  sort(temp.begin(), temp.end(), compare);

  vector<bool> prime(std::stoi(temp) + 1);

  //cout << stoi(temp) << endl;
  prime[0] = false;
  for (long long i = 1; i < prime.size(); i++)
  {
    prime[i] = isPrime(i);
  }
  //cout << "chk1" << endl;
  //int num = std::stoi(numbers);

  string s, sub;

  s = numbers;

  sort(s.begin(), s.end());
  set<int> primes;
  int l = s.size();
  do
  {
    for (int i = 1; i <= l; i++)
    {
      sub = s.substr(0, i);
      //  cout << "chk2" <<  " " << sub<<  endl;
      if (prime[std::stoi(sub)])
      {
        primes.insert(std::stoi(sub));
      }
    }
  } while (next_permutation(s.begin(), s.end()));

  //cout << primes.size();
  answer = primes.size();
  return answer;
}
```
해당 풀이가 프로그래머스 상에서 가장 많은 추천을 받은 풀이입니다. 엄청난 추천 수를 받았는데 평소와는 느낀 점이 조금 다릅니다. 우선 **해당 알고리즘에서 핵심은 next_permutation()이라는 C++ algorithm 라이브러리 내에 순열 메소드를 사용했다는 점**입니다만, **그 외에 코드는 썩 효율적이지 못한 부분도 존재한다고 생각**합니다. 우선 next_permutation()이 무엇인지 알아보겠습니다.

## next_permutation()이란?
> **next_permutation()은 C++ algorithm 내에 존재하는 순열 기능을 제공하는 메소드**입니다. 기본적으로 next_permutation()은 **인자로 iterator를 받기 때문에 vector 뿐만 아니라 string 타입 변수의 순열도 구해낼 수 있습니다.** 이 메소드는 **더 큰 순열로 재배열할 수 있으면 반복하여 구해내는 로직**을 가지고 있어서 **앞에 이미 큰 원소들이 배치되어 있으면 모든 순열의 경우의 수를 구할 수 없습니다.** 즉, **오름차순으로 정렬되어있는 상태에서 next_permutation()을 사용해야 모든 경우의 수를 구할 수 있습니다.**
``` cpp
string s = "1234";
do {
	cout << s << " ";
} while (next_permutation(s.begin(), s.end()));
```
위와 같이 사용하면 1,2,3,4를 활용한 모든 순열이 출력되게 됩니다.

다시 위 알고리즘으로 돌아와서 간략한 로직을 설명하자면, isPrime()은 이름에서 유추할 수 있듯이 소수인지 확인해주는 메소드입니다. **이 내부에서는 sqrt()를 활용해서 조금 더 효율적으로 소수를 찾을 수 있도록 구현**하였습니다.
solution() 부분을 보면 다소 이해하기 어려울 수 있는데 각각의 변수 및 로직에 대하여 간략히 설명하자면 **prime이라는 vector<bool>이 있는데 이 벡터에는 0부터 numbers로 만들 수 있는 최대의 숫자까지 인덱스를 가지게 구현되어 있고 각각의 인덱스에 해당하는 값은 해당 인덱스가 소수인지 아닌지에 따라서 bool 값을 가지게 구현**되어 있습니다. 즉 prime[3]을 예로들면 3은 소수이기 때문에 true라는 값을 가지고 있습니다. 간단히 말해서 **numbers로 만들 수 있는 숫자 범위 내의 모든 숫자에 대한 소수 여부 체크를 미리 해놓고 담아놓은 변수**라고 생각하시면 될 것 같습니다.
그렇다면 추후에 이 벡터의 인덱스에 접근해서 소수 여부를 체크하기 쉽겠죠?

s라는 string은 numbers 값을 가지고 있고 오름차순으로 정렬합니다. 왜냐하면 **next_permutation()을 활용하기 위해서 오름차순으로 정렬해놓은 것**이죠. 이렇게 구해진 **모든 순열의 경우의 수에서 구할 수 있는 모든 부분 문자열을 구하고 해당 부분 문자열을 int형으로 바꿔서 아까 미리 소수를 체크해놓은 prime 벡터에 인덱스로 접근하여 소수 여부를 체크**합니다. **소수가 맞다면 primes라는 set 자료 구조에 해당 값을 넣어 소수의 개수를 카운팅**하는 방식으로 이루어져 있습니다. **제 알고리즘과 다른 큰 차이점이라면 순열의 경우의 수를 구하기 위해서 저는 재귀 함수를 활용했다면 위 알고리즘은 next_permutation() 메소드를 활용**했다고 볼 수 있겠습니다.
  

  
## 최적화한 나의 풀이
최적화한 풀이가 처음에 본인이 푼 풀이와 동일하여서 이하 설명은 생략하도록 하겠습니다.

## References
> https://notepad96.tistory.com/entry/C-%EC%88%9C%EC%97%B4Permutation-nextpermutation  
https://blockdmask.tistory.com/79