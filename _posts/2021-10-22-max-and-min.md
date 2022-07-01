---
title: 연습 문제_ 문자열 - Level2 - 최댓값과 최솟값
categories: [Computer science, Algorithm]
tags: [level2, programmers, string, 연습 문제, 최댓값과 최솟값, 문자열, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 연습문제 - [최댓값과 최솟값](https://programmers.co.kr/learn/courses/30/lessons/12939) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <sstream>
using namespace std;

string solution(string s) {
    string answer = "";
    int num = 0, min = 0, max = 0;
    stringstream ss(s);
    ss >> num;
    min = max = num;
    
    while(ss >> num) {
        if(num<min)
            min = num;
        if(num>max)
            max = num;
    }
    answer = to_string(min) + " " + to_string(max);
    return answer;
}
```

바로 전날에 문자열 토크나이징과 관련된 카카오 Level2 문제를 풀었어서 그런지 이 문제는 정말 금방 풀렸습니다. 제가 앞선 포스팅에서 말씀드린 것처럼 **cpp에서 문자열을 토크나이징 하는 방법에는 크게 2가지 있습니다. 저는 그중에서도 조금 더 간단한 stringstream을 이용**해봤습니다.

## stringstream이란?
> **stringstream은 주어진 문자열에서 필요한 정보를 빼낼 때 유용하게 사용**됩니다. **특히나 내가 전달해주는 자료형과 일치하는 부분만 공백, '\n'을 무시하고 추출합니다. stringstream은 <sstream\>이라는 헤더 파일 내에 구현되어 있습니다.**
``` cpp
stringstream ss(input);
```
**우선 stringstream을 사용하기 위해서 위처럼 객체를 선언해줘야 합니다. 생성자를 이용해서 input이라는 string 값을 가지는 stringstream 객체를 생성한 것인데 위에처럼 선언 시에 해도 되고 아니면 아래와 같이 추후에 stringstream에 들어가있는 string 값을 바꿔주거나 새로 넣어줄 수도 있습니다.**
``` cpp
ss.str(input);
```
근데 여기서 주의할 것이 같은 **stringstream 객체를 어느정도 쓰다가 ss.str(input);과 같이 새로운 string 값을 넣어준 뒤 바로 사용하려고 하면 이전 string 값으로 사용하다가 남은 Flag들이 그대로 저장되어 있어서 올바르게 동작하지 않을 수도 있으니 동일한 stringstream 객체에 string 값만 바꿔가면서 계속 사용한다고 한다면 새로운 string 값을 사용하기 전에 반드시 아래와 같이 클리어를 한번 해주고 사용**합시다.
``` cpp
ss.clear();
```
그렇다면 이제 stringstream을 가지고** 어떻게 공백,'\n'을 기준으로 문자열을 토크나이징 할 것인가**에 대해 알아볼까요?
토큰을 담을 변수를 가지고 아래와 같이 사용해주면 됩니다.
``` cpp
ss >> name;
```
**위처럼 사용하면 1개의 토큰만 추출되는 것이고 이걸 문자열 끝까지 반복하려면 아래와 같이** 하면 됩니다.
``` cpp
while(ss >> name) {
	cout << name << endl;
}
```
만약에 **name이 string이면 string 타입에 맞는, 즉 문자열 내에서도 문자와 관련된 값들이 나타나는 곳까지만 추출된다는 것을 꼭 기억**합시다.
즉, 만약에 **주어진 string이 "13 256 .25 a"이고 int형 데이터 타입을 가지는 num으로 추출을 한다면 위 반복문은 13과 256만 추출하고 끝나는 것 입니다. float num이라면 13, 256, 0.25로 추출이 됩니다.
string이라면 "13", "256", ".25", "a"로 추출이 됩니다.**

## 다른 사람의 풀이1
``` cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

string solution(string s) {
    string answer = "";
    string sTemp = "";
    vector<int> vecInteger;

    for (int i = 0; i < s.size(); i++)
    {
        if (s[i] == ' ')
        {
            vecInteger.push_back(stoi(sTemp));
            sTemp.clear();
            continue;
        }

        sTemp += s[i];
    }

    vecInteger.push_back(stoi(sTemp));

    sort(vecInteger.begin(), vecInteger.end());

    answer += to_string(vecInteger.front());
    answer += ' ';
    answer += to_string(vecInteger.back());

    return answer;
}
```
위 알고리즘이 추천을 많이 받은 알고리즘인데, 제 개인적인 견해입니다만 제 알고리즘처럼 **stringstream을 사용하는 것이 훨씬 더 간단해보이고 좋은 것 같습니다. 그럼에도 다른 개발자의 알고리즘을 이해하고 체득한다는 건 정말 큰 의미가 있으니까 같이 한번 알아봅시다.**

**문자열의 인덱스 접근을 하면서 공백이 나타나면 지금까지 sTemp에 쌓인 string을 Integer로 바꿔준 뒤 vecInteger라는 벡터에 차곡차곡 쌓는 방식입니다. 그 후 이걸 정렬해서 min값과 max값을 찾아내는 방식**입니다. 코드 수는 조금 더 길어보이지만 **위 알고리즘이 이해하기 좋네요! 역시 사람들이 추천하는 데는 이유가 있는 것 같습니다.**

## 최적화한 나의 풀이
최적화한 풀이가 처음에 본인이 푼 풀이와 동일하여서 이하 설명은 생략하도록 하겠습니다.
