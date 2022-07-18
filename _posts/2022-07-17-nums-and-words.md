---
title: 카카오_ 문자열 - Level1 - 숫자 문자열과 영단어
categories: [Computer science, Algorithm]
tags: [level1, programmers, string, 카카오, 숫자 문자열과 영단어, 문자열, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 카카오 - [숫자 문자열과 영단어](https://school.programmers.co.kr/learn/courses/30/lessons/81301) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>

using namespace std;

int solution(string s) {
    int answer = 0;
    vector<string> num = {"zero", "one", "two", "three", "four", "five", "six", "seven", "eight", "nine"};
    
    for(int i=0; i<num.size(); i++) {
        while(s.find(num[i]) != string::npos) {
            s.replace(s.find(num[i]), num[i].size(), to_string(i));
        }
    }
    
    return answer = stoi(s);
}
```

이 문제는 Level1이기 때문에 어렵지 않게 풀 수 있었습니다. 어떻게하면 최적의 알고리즘을 구할 수 있을지 먼저 고민하였고 그 결과 위처럼 풀이하였습니다. 위 알고리즘에서 핵심 메소드는 find와 replace입니다. str.find(str1)를 하면 str에서 str1 부분을 찾아 해당 부분이 시작하는 인덱스 번호를 리턴해줍니다. 여기서 주의할 것은 이터레이터가 아닌 인덱스 번호입니다. 또한 찾지 못할 경우에는 string::npos를 리턴함을 잊어서는 안됩니다.  
그다음으로 위 로직에서 중요한 메소드는 str.replace(시작 인덱스, 길이, 대체할 문자열)입니다. str의 시작 인덱스로부터 길이만큼의 부분 문자열을 대체할 문자열로 대체해주는 함수입니다. 이 두가지 함수만 잘 활용한다면 손쉽게 위 문제를 해결할 수 있습니다.

## 다른 사람의 풀이1
``` cpp
#include <bits/stdc++.h>
using namespace std;

int solution(string s) {
    s = regex_replace(s, regex("zero"), "0");
    s = regex_replace(s, regex("one"), "1");
    s = regex_replace(s, regex("two"), "2");
    s = regex_replace(s, regex("three"), "3");
    s = regex_replace(s, regex("four"), "4");
    s = regex_replace(s, regex("five"), "5");
    s = regex_replace(s, regex("six"), "6");
    s = regex_replace(s, regex("seven"), "7");
    s = regex_replace(s, regex("eight"), "8");
    s = regex_replace(s, regex("nine"), "9");    
    return stoi(s);
}
```
처음보고 C++ 코드가 아닌 줄 알았습니다. 생전 처음 보는 헤더 파일과 메소드였습니다. 하지만 메소드 명을 보고 정규표현식과 관련된 함수라는 것은 눈치챌 수 있었습니다. 그래서 지금부터는 regex_replace() 메소드에 대하여 알아보려고 합니다.

## regex_replace()란?
> regex_replace()는 대상 문자열에서 문자열을 검색해 치환하기 위해서 사용할 수 있습니다. 이 함수는 \<regex>에 들어있으며 사용 방법은 아래와 같습니다.
```
regex_replace.(대상 문자열, regex(정규식), 치환 문자열)
```
이렇게 함수를 사용하면 대상 문자열에서 정규식에 지정한 문자열과 일치하는 문자열을 모두 치환 문자열로 대체한 뒤 바뀐 대상 문자열을 리턴해 줍니다.

## 최적화한 나의 풀이
최적화한 풀이가 처음에 본인이 푼 풀이와 동일하여서 이하 설명은 생략하도록 하겠습니다.

## References
> https://ponyozzang.tistory.com/678  
