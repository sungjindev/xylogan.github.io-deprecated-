---
title: 카카오_ 문자열 - Level1 - 신고 결과 받기
categories: [Computer science, Algorithm]
tags: [level1, programmers, string, 카카오, 신고 결과 받기, 문자열, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 카카오 - [신고 결과 받기](https://school.programmers.co.kr/learn/courses/30/lessons/92334) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <set>
#include <sstream>
#include <iostream>
#include <unordered_map>
using namespace std;

vector<int> solution(vector<string> id_list, vector<string> report, int k) {
    vector<int> answer(id_list.size());
    //일단 report에서 중복 제거
    vector<int> call_count(id_list.size());
    set<string> s1(report.begin(), report.end());
    // unodered_map<string,string> find_caller_map;
    vector<pair<string,string>> caller_callee;
    unordered_map<string,int> name_map;
    for(int i=0; i<id_list.size(); i++) {   //빠른 연산을 위해 언오더드 맵에 이름, 인덱스 저장 
        name_map[id_list[i]] = i;
    }
    
    for(auto it=s1.begin(); it!=s1.end(); ++it) {
        stringstream ss(*it);
        string caller,callee;
        ss >> caller;
        ss >> callee;
        // find_caller_map[callee] = caller;
        caller_callee.push_back(make_pair(caller, callee));
        call_count[name_map[callee]]++;
    }
    
    for(int i=0; i<call_count.size(); i++) {
        if(call_count[i] >= k) {
            for(auto name_index: name_map) {
                if(name_index.second == i) {
                    for(auto cc: caller_callee) {
                        if(cc.second == name_index.first) {
                            answer[name_map[cc.first]]++;
                        }
                    }
                }
            }
        }
    }

    return answer;
}
```

이 문제는 문자열 관련 문제로 Level1임에도 불구하고 시간도 꽤 오래 걸리고 소스 코드도 풀면 풀수록 점점 길어져서 많이 당황스러웠던 문제입니다. 그리고 시간 복잡도를 고려안할 수가 없어서 최대한 빠른 자료 구조를 사용하고자 다양하게 시도해봤더니 헤더 파일 개수만 엄청 늘어났습니다. 그러던 와중 문제를 다 풀고나서 다른 사람 풀이와 비교해보니 로직 자체가 굉장히 비슷해서 놀랐습니다. 이 문제는 report에서 먼저 중복을 제거해주고 문자열 토크나이징 해준 뒤에 신고한 사람과 신고 받은 사람을 잘 묶어 문제 조건에 맞게 사용해주면 쉽게 해결할 수 있습니다.

## 다른 사람의 풀이1
``` cpp
#include <bits/stdc++.h>
#define fastio cin.tie(0)->sync_with_stdio(0)
using namespace std;

vector<int> solution(vector<string> id_list, vector<string> report, int k) {
    // 1.
    const int n = id_list.size();
    map<string, int> Conv;
    for (int i = 0; i < n; i++) Conv[id_list[i]] = i;

    // 2.
    vector<pair<int, int>> v;
    sort(report.begin(), report.end());
    report.erase(unique(report.begin(), report.end()), report.end());
    for (const auto& s : report) {
        stringstream in(s);
        string a, b; in >> a >> b;
        v.push_back({ Conv[a], Conv[b] });
    }

    // 3.
    vector<int> cnt(n), ret(n);
    for (const auto& [a, b] : v) cnt[b]++;
    for (const auto& [a, b] : v) if (cnt[b] >= k) ret[a]++;
    return ret;
}
```

위 풀이가 프로그래머스 상에서 가장 많은 추천을 받은 풀이입니다. 제 풀이와 로직 자체는 굉장히 비슷합니다. 저는 set 자료 구조를 활용해서 중복을 제거해준 반면에 이 풀이에서는 unique와 erase를 활용하여 중복을 제거해 줬습니다. 제가 생각했을 때 이 풀이가 제 풀이보다 간결하고 효율적일 수 있었던 이유는 처음부터 사용자들의 이름을 index와 매핑시킨 뒤 쭉 index로만 접근했다는 점입니다. 저는 이름과 index를 일관성있게 매핑시키지 않아서 이 두 관계를 판단하고 변환하는데 많은 비용을 들였습니다. 이 부분 외에는 로직 자체가 굉장히 똑같습니다.

## 최적화한 나의 풀이
최적화한 풀이가 다른 사람이 푼 풀이와 동일하여서 이하 설명은 생략하도록 하겠습니다.
