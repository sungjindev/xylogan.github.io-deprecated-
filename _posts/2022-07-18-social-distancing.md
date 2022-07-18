---
title: 카카오_ 완전탐색 - Level2 - 거리두기 확인하기
categories: [Computer science, Algorithm]
tags: [level2, programmers, exhaustive search, 카카오, 거리두기 확인하기, 완전탐색, 알고리즘, 코딩 테스트, 프로그래머스]
---

**!본 포스팅은 프로그래머스 코딩테스트 카카오 - [거리두기 확인하기](https://school.programmers.co.kr/learn/courses/30/lessons/81302) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <iostream>
using namespace std;

int vector_to_array(vector<string> v1, char arr[][5]) {
    for(int i=0; i<5; i++) {
        for(int j=0; j<5; j++) {
            arr[i][j] = v1[i][j];
        }
    }
    return 0;
}

int calc(char arr[][5]) {
    for(int i=0; i<5; i++) {
        for(int j=0; j<5; j++) {
            if(arr[i][j] == 'P') {
                if(i+1 <= 4) {
                    if(arr[i+1][j] == 'P')
                        return 0;
                }
                if(i-1 >= 0) {
                    if(arr[i-1][j] == 'P')
                        return 0;
                }
                if(j+1 <= 4) {
                    if(arr[i][j+1] == 'P')
                        return 0;
                }
                if(j-1 >= 0) {
                    if(arr[i][j-1] == 'P')
                        return 0;
                }
                if(i+1<=4 && j+1<=4) {
                    if(arr[i+1][j+1] == 'P') {
                        if(arr[i+1][j] != 'X')
                            return 0;
                        if(arr[i][j+1] != 'X')
                            return 0;
                    }
                }
                if(i-1>=0 && j+1<=4) {
                    if(arr[i-1][j+1] == 'P') {
                        if(arr[i-1][j] != 'X')
                            return 0;
                        if(arr[i][j+1] != 'X')
                            return 0;    
                    }
                }
                if(i-1>=0 && j-1>=0) {
                    if(arr[i-1][j-1] == 'P') {
                        if(arr[i][j-1] != 'X')
                            return 0;
                        if(arr[i-1][j] != 'X')
                            return 0;
                    }
                }
                if(i+1<=4 && j-1>=0) {
                    if(arr[i+1][j-1] == 'P') {
                        if(arr[i][j-1] != 'X')
                            return 0;
                        if(arr[i+1][j] != 'X')
                            return 0;
                    }
                }
                if(i+2<=4) {
                    if(arr[i+2][j] == 'P') {
                        if(arr[i+1][j] != 'X')
                            return 0;
                    }
                }
                if(j+2<=4) {
                    if(arr[i][j+2] == 'P') {
                        if(arr[i][j+1] != 'X')
                            return 0;
                    }
                }
                if(i-2>=0) {
                    if(arr[i-2][j] == 'P') {
                        if(arr[i-1][j] != 'X')
                            return 0;
                    }
                }
                if(j-2>=0) {
                    if(arr[i][j-2] == 'P') {
                        if(arr[i][j-1] != 'X')
                            return 0;
                    }
                }
            }
        }
    }
    return 1;
}

vector<int> solution(vector<vector<string>> places) {
    vector<int> answer;
    for(int i=0; i<places.size(); i++) {
        char arr[5][5];        
        vector_to_array(places[i], arr);
        answer.push_back(calc(arr));
    }     
    
    return answer;
}
```

이 문제는 완전 탐색 문제로, 문제를 풀고 난 뒤에 다른 사람들의 풀이들을 보니 DFS나 BFS를 활용해서 풀이하신 분들이 많았습니다. 저와 같은 경우에는 사람 "P"의 입장에서 가능한 모든 경우의 수를 조건문을 활용해서 풀이하였는데 이렇게 풀고보니 정답은 맞았지만 굉장히 지저분해서 아쉬움이 많이 남습니다. 문제의 조건을 코드로 옮겨 적은 느낌이라 코드를 보시면 쉽게 이해가 가능해서 별도의 풀이는 적지 않겠습니다.

## 다른 사람의 풀이1
``` cpp
#include <string>
#include <vector>
#include <iostream>

using namespace std;

bool is_valid_place(const vector<string>& place)
{
    constexpr size_t N = 5;

    vector<vector<int>> is_in_use(
        N,
        vector<int>(N, false)
    );

    int di[] = {1,-1,0,0};
    int dj[] = {0,0,1,-1};

    for(size_t i=0; i!=N; ++i)
        for(size_t j=0; j!=N; ++j)
            if(place[i][j] == 'P'){
                for(size_t k=0; k!=4; ++k){
                    size_t moved_i = i + di[k];
                    size_t moved_j = j + dj[k];

                    if(moved_i >= N || moved_j >= N)
                        continue;

                    switch(place[moved_i][moved_j]){
                        case 'P':
                            return false;
                        case 'O':
                            if(is_in_use[moved_i][moved_j])
                                return false;

                            is_in_use[moved_i][moved_j] = true;
                            break;
                        case 'X':
                            break;
                    }
                }
            }

    return true;
}

vector<int> solution(vector<vector<string>> places) {
    vector<int> answer(5);
    for(size_t i=0; i!=5; ++i)
        answer[i] = is_valid_place(places[i]);
    return answer;
}
```

위 풀이가 프로그래머스 상에서 가장 많은 추천을 받은 풀이입니다. 위 풀이를 간단히 이해하기 좋게 설명해보자면, is_in_use라는 2차원 벡터에 모두 false값을 채워 만들고 'P' 원소를 가지는 좌표에서 사방으로 한 칸씩 움직이며 'P'를 만나는지 'O'를 만나는 지에 따라 문제의 조건을 해결합니다. 'P'를 만나는 경우에는 당연히 거리두기에 실패한 것이고 'O'를 만나는 경우에는 누군가 'O'에 1칸 거리에 있다는 것을 표현하기 위해 is_in_use를 true로 바꿔줍니다. 그 후 다른 'P'가 위처럼 사방으로 한 칸씩 움직이다가 누군가 왔다간 흔적(is_in_use가 true)를 발견한다면 그것 또한 거리두기에 실패한 것으로 처리할 수 있게 됩니다. 한줄로 요약하면, 'P' 원소를 기준으로 사방으로 한 칸씩 움직여서(한 칸 움직인 뒤 만난 원소가 'O'일때만 움직일 수 있음) 인접한 'P'와 만날 수 있는 지를 검사하여 거리두기를 검사한 로직입니다.

## 최적화한 나의 풀이
최적화한 풀이가 다른 사람이 푼 풀이와 동일하여서 이하 설명은 생략하도록 하겠습니다.
