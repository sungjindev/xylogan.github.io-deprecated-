---
title: [백준 1463번]_ DP - 실버3 - 1로 만들기
categories: [Computer science, Algorithm]
tags: [실버3, baekjoon online judge, dynamic programming, 1463번, 1로 만들기, DP, 알고리즘, 코딩 테스트, 백준]
---

**!본 포스팅은 백준 코딩테스트 1463번 - [1로 만들기](https://www.acmicpc.net/problem/1463) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <iostream>
using namespace std;

int arr[3000000] = {0, };

void calc(int n);

int main(void) {
    int n=0;
    cin >> n;
    
    calc(n);

    return 0;
}

void calc(int n) {
    for(int i=1; i<n; i++) {
        if(!arr[i+1]) {
            arr[i+1] = arr[i]+1;
        } else if(arr[i+1] > arr[i]+1) {
            arr[i+1] = arr[i]+1;
        }
        
        if(!arr[i*2]) {
            arr[i*2] = arr[i]+1;
        } else if(arr[i*2] > arr[i]+1) {
            arr[i*2] = arr[i]+1;
        }
        
        if(!arr[i*3]) {
            arr[i*3] = arr[i]+1;
        } else if(arr[i*3] > arr[i]+1) {
            arr[i*3] = arr[i]+1;
        }
    }
    cout << arr[n];
}
```

이 문제는 Dynamic Programming(DP)를 활용한 문제입니다. DP란 동적 계획법이란 뜻으로 복잡한 문제를 간단한 여러 문제로 나눠서 푸는 방식을 말하는데 이와 관련해서는 잠시 뒤 더욱 자세히 알아보도록 하겠습니다. 우선 제가 푼 풀이를 보면 저는 Bottom-Up 방식으로 풀이하였습니다. 즉 문제에서 구해야하는 것이 n에 대한 최소 연산 횟수라면 저는 clac() 메소드 안에 for문을 통해 1일 때부터 n이 될때까지 차근 차근 구합니다. 이때 DP의 핵심인 Memoization(이전에 연산한 데이터를 기록 및 저장해 둠)을 활용하여 중복된 연산이 반복되지 않고 최적의 해가 산출될 수 있도록 if문과 정적 배열을 활용하여 처리해줬습니다. 

## Dynamic Programming(DP)란?
> Dynamic Programming(DP)는 동적 계획법이란 뜻으로 복잡한 문제를 간단한 여러 문제로 나눠서 푸는 방식을 말합니다. 이렇게 하면 코드 가독성이 올라가는 장점도 있지만 기본적으로 간단한 여러 문제로 부분 부분 연산을 하면서 앞으로 쓰일만한 연산 후 결과값은 배열과 같은 곳에 저장해주는 Memoization 방식을 활용해서 최종적으로 연산의 횟수를 줄이며 최적의 해를 산출할 수 있도록 해줍니다. 물론 모든 문제가 DP 방식으로 풀었을 때 가장 효율적이진 않습니다. DP로 활용하기 위해서는 작은 문제로 나누어질 수 있어야하며 그렇게 작은 문제들의 연산이 최종 결과값을 구할 때 반복되어 연산되어야 한다면 Memoization을 활용하기 때문에 DP의 장점이 더욱 극대화될 수 있을 것입니다. 

## 다른 사람의 풀이1
``` cpp
#include <iostream>
#include <algorithm>
using namespace std;

int n;
int dp[1000002];

int solve(int k) {
    if(k<=0) return 1e9;
    if(k==1) return 0;
    if(dp[k]) return dp[k];
    dp[k] = 1e9;
    if(k%2==0) {
        dp[k] = min(dp[k], solve(k/2)+1);
    }
    if(k%3==0) {
        dp[k] = min(dp[k], solve(k/3)+1);
    }
    dp[k] = min(dp[k], solve(k-1)+1);
    return dp[k];
}

int main() {
    cin >> n;
    dp[2] = 1;
    dp[3] = 1;
    int ans = solve(n);
    cout << ans;
    return 0;
}
```

이 문제와 관련해서 여러 풀이를 찾아봤지만 제가 생각하기에는 위 풀이가 이해하기도 쉽고 가장 깔끔하게 짜여진 코드 같습니다. 기본적으로 저와 로직 자체는 거의 비슷하지만 가장 큰 차이점으로는 저는 Memoization을 위한 배열을 0으로 초기화 해준 반면에 위 풀이에서는 엄청나게 큰 수를 사용해서 min()을 구하는 연산에 방해가 되지 않게 처리하였습니다. 왜냐하면 저처럼 저장하는 배열의 초기값을 0으로 설정해버리면 min() 연산을 할 때 항상 0이 나오기 때문에 저처럼 if문을 통해 0일 경우를 따로 처리해주는 방식을 사용해야 하기 때문입니다. 또 하나 제 풀이와 다른 큰 차이점은 solve()라는 함수를 재귀적으로 호출하면서 구하고자 하는 N이라는 값부터 시작하여 1까지 Top-Down 방식을 채택하여 풀이했다는 점입니다. 이 외에는 대부분의 로직이 동일하여 부가적인 설명은 생략하도록 하겠습니다. 


## 최적화한 나의 풀이
최적화한 풀이가 다른 사람이 푼 풀이와 동일하여서 이하 설명은 생략하도록 하겠습니다.
