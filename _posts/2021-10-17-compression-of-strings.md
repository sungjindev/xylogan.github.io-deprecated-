---
title: 카카오_ 문자열 - Level2 - 문자열 압축
categories: [Computer science, Algorithm]
tags: [level2, programmers, string, 카카오, 문자열 압축, 문자열, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 카카오 - [문자열 압축](https://programmers.co.kr/learn/courses/30/lessons/60057) 풀이입니다.**

오늘은 **카카오 블라인드 채용 문제**를 준비해 봤습니다. 

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
using namespace std;

int solution(string s) {
    int answer = s.length();
    
    for(int i=1; i<=s.length()/2; i++) {
        string temp = "";
        int count = 1;
        int toggle = 0;
        int tempAnswer = s.length();
        
        for(int j=0; j+i<=s.length(); j += i) {
            if(temp != "" && temp == s.substr(j,i)) {
                count++;
                toggle = 1;
            }
            else {
                if(toggle == 1) {
                    tempAnswer = tempAnswer - (i*count-(i+1));
                    if(10<=count && count<=99)
                        tempAnswer += 1;
                    else if(100<=count && count <=999)
                        tempAnswer += 2;
                    else if(count == 1000)
                        tempAnswer += 3;
                    count = 1;
                }
                toggle = 0;
            }
            temp = s.substr(j,i);
        }
        
        if(toggle == 1) {
            tempAnswer = tempAnswer - (i*count-(i+1));
            if(10<=count && count<=99)
                tempAnswer += 1;
            else if(100<=count && count <=999)
                tempAnswer += 2;
            else if(count == 1000)
                tempAnswer += 3;
        }
        
        if(tempAnswer < answer)
            answer = tempAnswer;
    }
    
    return answer;
}
```
코드가 평소보다 좀 길어보입니다.
그래보이긴 해도 나름대로 알아보기 쉽게 구성되어 있다고 생각합니다.
일단 이 문제는 문자열 압축이라는 문제로, **문자열을 토큰화 시켜서 분리한뒤에 인접한 토큰끼리 중복된 토큰이 나타나면 Count를 1씩 증**가시켜서 개수를 세주는 로직을 사용해야 합니다. 다음 다른 사람의 풀이에서도 나타나겠지만 여기에 있어서 **크게 2가지로 나뉘는데, 실제로 압축된 총 문자열을 구해서 최종적으로 그 길이를 구하는 방법과 아니면 저처럼 압축된 문자열을 직접 구하지 않고 그때그때 마다 압축되면서 원래 문자열의 총 길이에서 얼만큼 줄어드는지 길이적인 관점에서만 계산을 하는 방법**, 이렇게 총 2가지가 있을 것 같습니다.
**저는 효율성 측면이 걱정이 되서 직접 문자열을 구하진 않고 길이적인 관점에서만 계산**을 했습니다. **문자열이 압축이 되면 아래와 같은 일반화된 식에서 나오는 값으로 문자열이 압축**됩니다.

```cpp
tempAnswer = tempAnswer - (i*count-(i+1));
```
근데 여기서 하나 주의할 점이 있습니다. **그것은 바로 위 일반화된 식은 Count 값이 1자리수 일때만 통한다는 것**입니다. 
**그래서 저는 아래와 같이 후처리**를 해줬습니다.
```cpp
if(10<=count && count<=99)
	tempAnswer += 1;
else if(100<=count && count <=999)
	tempAnswer += 2;
else if(count == 1000)
	tempAnswer += 3;
```
어? 그런데 지금 이 포스팅을 적다가 보니까 위 식에서 **(i+1)이 부분에서 1부분만 Count의 자리수로 일반화 시켜주면 코드가 훨씬 간결**해질 수 있겠다라는 생각이 듭니다.
그리고 이 코드에서 **비효율적인 부분이 하나 있는데, 바로 toggle 변수를 썼다는 점**입니다. 급한 마음에 toggle 변수를 추가해서 구현을 했는데 **추후 최적화한 나의 풀이에서 이 부분에 대한 수정이 있으니까 꼭 끝까지 읽어보시길 바랍니다.**


## 다른 사람의 풀이1
``` cpp
#include <string>
#include <vector>
#include <iostream>

using namespace std;

vector<string> convert(string s, int n)
{
    vector<string> v;
    for(int i = 0 ; i <s.length() ; i+=n)
    {
        v.push_back(s.substr(i,n));
    }
    return v;
}

int solution(string s) {
    int answer = 0;
    vector<string> tok;
    string before;
    int cnt = 1;
    int min = s.length();
    string str="";
    for(int j = 1 ; j <= s.length()/2 ; j++)
    {
        tok = convert(s,j);
        str = "";
        before = tok[0];
        cnt = 1;
        for(int i = 1 ; i < tok.size() ; i++)
        {
            if(tok[i] == before) cnt++;
            else
            {
                if(cnt != 1) str += to_string(cnt);
                str += before;
                before = tok[i];
                cnt = 1;
            }
        }
        if(cnt != 1)str += to_string(cnt);
        str += before;  
        min = min<str.length() ? min : str.length();
    }
    cout<<str;

    return min;
}
```
위 알고리즘의 전반적인 큰 틀은 비슷하지만 앞서 말씀드린 것처럼 저와 다른 방식인, **직접 압축된 문자열을 구하는 방식**입니다.
일단 제 코드보다 읽기도 좋고 더 이해하기 쉽게 잘 짜여진 코드 같습니다.
큰 흐름 자체는 저와 굉장히 유사하고 **토큰 비교해주는 로직 내에서 직접 압축된 문자열을 str에 구하고 있다는 점만 유의**해서 봐주시면 될 것 같습니다.

## 최적화한 나의 풀이
``` cpp
#include <string>
#include <vector>
using namespace std;

int solution(string s) {
    int answer = s.length();
    
    for(int i=1; i<=s.length()/2; i++) {
        string temp = "";
        int count = 1;
        int tempAnswer = s.length();
        
        for(int j=0; j+i<=s.length(); j += i) {
            if(temp != "" && temp == s.substr(j,i))
                count++;
            else {
                if(count != 1) {
                    tempAnswer = tempAnswer - (i*count-(i+1));
                    if(10<=count && count<=99)
                        tempAnswer += 1;
                    else if(100<=count && count <=999)
                        tempAnswer += 2;
                    else if(count == 1000)
                        tempAnswer += 3;
                    count = 1;
                }
            }
            temp = s.substr(j,i);
        }
        
        if(count != 1) {
            tempAnswer = tempAnswer - (i*count-(i+1));
            if(10<=count && count<=99)
                tempAnswer += 1;
            else if(100<=count && count <=999)
                tempAnswer += 2;
            else if(count == 1000)
                tempAnswer += 3;
        }
        
        if(tempAnswer < answer)
            answer = tempAnswer;
    }
    
    return answer;
}
```
제 풀이를 다시 한번 리팩터링한 코드입니다. 뭐 크게 다를 것은 없습니다. 앞서 말씀드린 것처럼 **toggle을 굳이 쓸 필요가 없어서 제거**해봤습니다. **그리고 압축된 문자열의 길이를 구하는 일반화된 식에서 (i+1) 이 부분에 1은 Count의 자리수로 일반화 시키면 더 효율적인 코드**가 나올 수 있을 것 같습니다.