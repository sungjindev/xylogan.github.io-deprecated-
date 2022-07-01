---
title: 카카오_ 문자열 - Level2 - 오픈채팅방
categories: [Computer science, Algorithm]
tags: [level2, programmers, string, 카카오, 오픈채팅방, 문자열, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 카카오 - [오픈채팅방](https://programmers.co.kr/learn/courses/30/lessons/42888) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <map>
#include <cstring>
using namespace std;

vector<string> solution(vector<string> record) {
    vector<string> answer;
    map<string, string> m;  //map의 first가 uid, second가 nickname
    string uid = "";
    string name = "";
    string result = "";
    char* str_buff = new char[1000];
    
    for(int i=0; i < record.size(); i++) {
        strcpy(str_buff, record[i].c_str());
        char* tok = strtok(str_buff, " ");
        if(string(tok) == "Change" || string(tok) == "Enter") {
            tok = strtok(nullptr, " ");
            uid = string(tok);
            tok = strtok(nullptr, " ");
            name = string(tok);
            m[uid] = name;
        }
    }
    
    for(int i=0; i < record.size(); i++) {
        result = "";
        strcpy(str_buff, record[i].c_str());
        char* tok = strtok(str_buff, " ");
        if(string(tok) == "Enter") {
            tok = strtok(nullptr, " ");
            result += m[string(tok)];
            result += "님이 들어왔습니다.";
            answer.push_back(result);
        }
        else if(string(tok) == "Leave") {
            tok = strtok(nullptr, " ");
            result += m[string(tok)];
            result += "님이 나갔습니다.";
            answer.push_back(result);
        }
    }    
    
    return answer;
}
```

우선 이 문제를 이해하고 일반화된 식 혹은 알고리즘을 찾는 것은 그렇게 어렵지 않을 겁니다. 오픈채팅방에 사용자들이 들어오거나 나가면서 닉네임을 바꿀 수 있게 되는데 **가장 나중에 변경된 닉네임으로 최종적인 접속 로그가 찍히는 방식**입니다. 즉 마지막에 각 userId별로 최종적으로 어떤 닉네임을 가지게 되는지가 관건입니다. 그 후에는 그대로 출력만 해주면되서 어렵지는 않습니다.
저는 일단 **각 userId와 nickname을 엮어주고 빠른 시간으로 효율적으로 탐색하기 위해 map이라는 자료 구조를 사용**했습니다. 이전 포스팅에서도 유용하게 사용했던 알고리즘인데 오랜만에 보니까 간단히 정리하겠습니다.

## map이란?
> **map은 Key와 Value 쌍으로 이루어진 자료 구조이며 key값을 주면 value 값을 반환**하게 됩니다. map은 <map\>이라는 헤더파일 내에 존재합니다. 또한 **map은 중복을 허용하지 않으며 c++ map의 내부는 검색, 삽입, 삭제가 O(logN)인 레드블랙트리로 구현되어 있습니다. 매우 빠릅니다.**
map의 선언은 아래와 같습니다.
``` cpp
map<key, value> map1;
```
map에서 <key, value> 쌍을 삽입하는 것은 아래와 같이 합니다.
``` cpp
map1[key] = value;
혹은
map1.insert(make_pair(key, value));
```

이제 그 다음 알고리즘 내 핵심은 **공백을 기준으로한 문자열 토큰화**입니다. 공백을 기준으로 문자열을 잘라야 하는데 C**PP기준 이 부분에 있어서는 크게 2가지의 방법이 존재**하는 것 같습니다. **저는 그 중에서 cstring 헤더파일 내의 strtok()를 이용**하였습니다.

## strtok()란?
> ``` cpp
char* strtok(char* str, char* delimiters);
```
이런 식으로 구성된 **토크나이징 함수**입니다. cpp에서는 **<cstring>이라는 헤더파일 내에 존재**하고 있습니다. **delimiters라고 하면 구분 문자를 뜻하는데 구분 문자를 기준으로 string을 쪼갠다고 생각**하면 됩니다. 우선 위 함수 원형에서 볼 수 있듯이 **매개변수로 char*를 받기 때문에 cpp에서 흔히 쓰는 string을 그대로 넘겨주는 것은 불가능**합니다. 그래서 s**tring을 C style처럼 char* 형으로 바꿔주는 작업이 필요한데 여기서 또 하나 중요한 것이 매겨변수로 넘겨주는 str에 실체가 있는 문자 배열이어야 한다는 점**. 즉, **문자열 리터럴(문자열 상수: 실체가 없음)은 안된다**는 것 입니다. 쉽게 이해하기 위해 첨언하자면 **strtok() 메소드 내부에서 토크나이징을 위해 delimeters가 위치한 부분의 문자를 '\0'으로 바꾸는 과정이 있는데 문자열 리터럴은 기본적으로 const가 걸려있기 때문에 이러한 변경이 안됩니다. 그래서 에러가 발생**합니다.
그래서 저는 아래와 같은 작업을 해줬습니다.
``` cpp
//strtok 매개변수로는 실체가 있는 문자 배열만이 가능하므로 문자 배열 선언
char* str_buff = new char[1000]; 
//우선 string 클래스를 c_str()를 통해 string을 char* 형으로 바꿔줍니다.
strcpy(str_buff, record[i].c_str());
```
근데 여기서 중요한 점이 있습니다.
**record[i].c_str()이 이미 char*인데 이걸 그대로 strtok()의 str 매개변수로 
바로 쓰면 되는데 왜 이렇게 strcpy()를 통해서 다른 문자 배열로 옮겨준 것 일까요?** **그 이유는 바로 c_str() 메소드는 char* 형을 리턴하는 것이 아니고 const char\* 형을 리턴하기 때문**입니다. 
이렇게 되면 앞서 말한 것처럼 **strtok()이 해당 문자열에 구분 문자를 '\0'으로 변경시키는 작업을 하지 못하기 때문에 const가 없는 또다른 문자 배열에 strcpy()를 했다**고 이해하시면 됩니다.
그 후 아래처럼 이제 본격적으로 strtok()를 사용해주면 되는데,
``` cpp
char* tok = strtok(str_buff, " ");
```
**strtok()의 반환형이 char*** 임에 주의해주시고, 또 하나 중요한 점이 **1번 strtok()를 하고 나서 그 다음 토큰을 찾을 때는 아래와 같이 null 혹은 nullptr을 준다는 것을 기억**합시다.
``` cpp
tok = strtok(nullptr, " ");
```
이렇게 **널 포인터를 주는 이유는 strtok 내부 동작 원리 자체가 null 값을 넘겨주면 이전에 찾은 구분자 뒤에서부터 찾도록 구현이 되어있기 때문**입니다. **만약에 토크나이징 하다가 더이상 자를 게 없는 문자열 끝에 도달하게 되면 strtok()는 null 값을 리턴하게 된다는 것도 알아두시면 좋을 것 같습니다.**
참고로 이 문제에서는 굳이 그것까지 고려해줄 필요가 없어서 따로 쓰지는 않았습니다.

## 다른 사람의 풀이1
``` cpp
#include <string>
#include <vector>
#include <sstream>
#include <iostream>
#include <map>
using namespace std;


vector<string> solution(vector<string> record) {
    vector<string> answer;
    string command;
    string ID;
    string uid;
   map<string,string> m;


    for(string input:record)
    {
        stringstream ss(input);
        ss>>command;
        ss>>uid;
        if(command=="Enter" || command=="Change")
        {
            ss>>ID;
            m[uid]=ID;
        }
    }

   for(string input:record)
    {
        stringstream ss(input);
        ss>>command;
        ss>>uid;
        if(command=="Enter")
        {
         ID=(m.find(uid)->second);

            string temp = ID+"님이 들어왔습니다.";
         answer.push_back(temp);

        }
      else if(command=="Leave")
      {
         ID=(m.find(uid)->second);
            string temp = ID+"님이 나갔습니다.";
         answer.push_back(temp);
      }
    }
    return answer;
}
```
위 알고리즘이 무려 추천 수를 17개나 받은 알고리즘인데 큰 흐름 자체는 저와 똑같습니다. 하나 다른점이라고 한다면 토크나이징 방식이 다릅니다. 제가 이번 포스팅 초반에 말했던 것처럼 **cpp 내 토크나이징 방법은 크게 2가지가 있다고 했는데 그 중에 하나는 제가 사용한 <cstring\> 내부에 strtok()이고 나머지 하나가 바로 위 풀이에서 사용된 것처럼 stringstream을 활용하는 것**입니다.

## stringstream이란?
> **stringstream은 주어진 문자열에서 필요한 정보를 빼낼 때 유용하게 사용**됩니다. **특히나 내가 전달해주는 자료형과 일치하는 부분만 공백, '\n'을 무시하고 추출합니다. stringstream은 <sstream\>이라는 헤더 파일 내에 구현되어 있습니다.**
``` cpp
stringstream ss(input);
```
**우선 stringstream을 사용하기 위해서 위처럼 객체를 선언해줍니다. 생성자를 이용해서 input이라는 string 값을 가지는 stringstream 객체를 생성한 것인데 위에처럼 선언 시에 해도 되고 아니면 아래와 같이 추후에 stringstream에 들어가있는 string 값을 바꿔주거나 새로 넣어줄 수도 있습니다.**
``` cpp
ss.str(input);
```
근데 여기서 하나 주의할 것이 같은 **stringstream 객체를 어느정도 쓰다가 ss.str(input);과 같이 새로운 string 값을 넣어준 뒤 바로 사용하려고 하면 이전 string 값으로 사용하다가 남은 Flag들이 그대로 저장되어 있어서 올바르게 동작하지 않을 수도 있으니 동일한 stringstream 객체에 string 값만 바꿔가면서 계속 사용한다고 한다면 새로운 string 값을 사용하기 전에 반드시 아래와 같이 클리어를 한번 해주고 사용**해줍시다.
``` cpp
ss.clear();
```
그렇다면 이제 stringstream을 가지고** 어떻게 공백,'\n'을 기준으로 문자열을 토크나이징 할 것인가**에 대해 알아볼까요?
되게 간단합니다. 토큰을 담을 변수를 가지고 아래와 같이 사용해주면 됩니다.
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

## 최적화한 나의 풀이
최적화한 풀이가 처음에 본인이 푼 풀이와 동일하여서 이하 설명은 생략하도록 하겠습니다.

## References
> https://kamang-it.tistory.com/entry/cstring%EB%AC%B8%EC%9E%90%EC%97%B4-%EC%9D%B4%EC%95%BC%EA%B8%B0-2-%EB%AC%B8%EC%9E%90%EC%97%B4%EC%9D%84-%ED%8A%B9%EC%A0%95-%EB%AC%B8%EC%9E%90%EC%97%B4%EB%A1%9C-%EC%9E%90%EB%A5%B4%EA%B8%B0-%EB%AC%B8%EC%9E%90%EC%97%B4-%ED%86%A0%ED%81%AC%EB%82%98%EC%9D%B4%EC%A7%95  
https://kimcoder.tistory.com/122  
https://blockdmask.tistory.com/382  
https://codingdog.tistory.com/entry/c-string-cstr-%ED%95%A8%EC%88%98-string%EC%9D%84-char-%EB%A1%9C-%EB%B0%94%EA%BF%94%EB%B4%85%EC%8B%9C%EB%8B%A4  
https://word.tistory.com/24  