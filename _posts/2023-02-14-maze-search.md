---
title: 백준 2178번_ 그래프 - 실버1 - 미로 탐색
categories: [Computer science, Algorithm]
tags: [실버1, baekjoon online judge, graph, 2178번, 미로 탐색, 그래프, 알고리즘, 코딩 테스트, 백준]
---

**!본 포스팅은 백준 코딩테스트 2178번 - [미로 탐색](https://www.acmicpc.net/problem/2178) 풀이입니다.**

## 처음으로 맞춘 풀이
``` cpp
#include <iostream>
#include <queue>
#include <string>
#include <vector>
using namespace std;

vector<vector<int>> v1;
vector<vector<int>> check;
queue<pair<int,int>> q1;
int xpos[4] = {1,0,0,-1};
int ypos[4] = {0,1,-1,0};
int n,m;

void bfs(int a, int b) {
    check[a][b]=1;
    q1.push({a,b});

    while(!q1.empty()) {
        for(int i=0; i<4; i++) {
            int x = q1.front().first+xpos[i];
            int y = q1.front().second+ypos[i];
            
            if(x<0 || y<0 || x>=n || y>=m)
                continue;
            
            if(!check[x][y] && v1[x][y]) {
                q1.push({x,y});
                check[x][y]=check[x-xpos[i]][y-ypos[i]]+1;
            }
        }
        q1.pop();
    }
    
    cout << check[n-1][m-1];
}

int main(void) {
    string line;
    
    cin >> n >> m;
    v1 = vector<vector<int>>(n,vector<int>(m,0));
    check = vector<vector<int>>(n,vector<int>(m,0));
    
    for(int i=0; i<n; i++) {
        cin >> line;
        for(int j=0; j<line.size(); j++) {
            v1[i][j] = line[j]-'0';
        }
    }
    
    bfs(0,0);
    
    return 0;
}
```

이 문제는 그래프 관련 문제로 이차원 배열 상에서 상하좌우로 움직이며 BFS를 활용하는 문제입니다. BFS 특징상 최단 거리가 보장되기 때문에 쓰기 좋습니다. 이차원 배열 상에서 상하좌우로 움직이는 코드를 짤 때 거의 정형화된 포맷이 있는데 그게 위 코드에 xpos, ypos를 활용한 for문입니다. 또한, visited를 체크하기 위해서 저는 check라는 이차원 벡터를 사용했는데 여기서도 단순히 방문 여부만을 체크하지 않고 방문하지 않은 노드의 값은 0으로 한 뒤 방문할 때마다 이전에 방문한 노드의 check 값에 1씩 더해가면서 카운터 변수 그 자체로 활용할 수 있도록 구현하였습니다. 


## 다른 사람의 풀이1
``` cpp
#include <iostream>
#include <queue>
#define MAX 101

using namespace std;

int N, M;                       // 미로 크기
int maze[MAX][MAX];             // 미로 표현용 2차원 배열
int visited[MAX][MAX];          // 방문 기록용 2차원 배열
int dist[MAX][MAX];             // 이동칸 기록용 2차원 배열

int x_dir[4] = {-1, 1, 0, 0};   // 상화좌우 x축 방향
int y_dir[4] = {0, 0, -1, 1};   // 상화좌우 y축 방향

queue<pair<int,int> > q;        // 탐색 좌표 저장용 queue

// 미로 경로 탐색
void bfs(int start_x, int start_y){         

    visited[start_x][start_y] = 1;          // 입력 받은 시작 좌표를 방문
    q.push(make_pair(start_x,start_y));     // queue 에 삽입
    dist[start_x][start_y]++;               // 시작 좌표까지 이동한 칸을 1로 지정

    // 모든 좌표를 탐색할 때까지 반복
    while (!q.empty()){

        // queue 의 front 좌표를, 현재 좌표로 지정
        int x = q.front().first;
        int y = q.front().second;

        // qeueu 의 front 좌표 제거
        q.pop();

        // 현재 좌표와 상하좌우로 인접한 좌표 확인
        for (int i=0; i<4; ++i){

            // 현재 좌표와 상하좌우로 인접한 좌표
            int x_new = x + x_dir[i];
            int y_new = y + y_dir[i];

            // 인접한 좌표가, 미로 내에 존재하는지, 방문한 적이 없는지, 이동이 가능한지 확인
            if ( (0 <= x_new && x_new < N) && (0 <= y_new && y_new < M) 
            && !visited[x_new][y_new] && maze[x_new][y_new] == 1 ){

                visited[x_new][y_new] = 1;              // 인접 좌표는 방문한 것으로 저장
                q.push(make_pair(x_new, y_new));        // 인접 좌표를 queue 에 삽입
                dist[x_new][y_new] = dist[x][y] + 1;    // 인접 좌표까지 이동한 거리 저장
            }
        }
    }
}

int main(){

    cin >> N >> M;                      // 미로 크기 입력

    for (int i=0; i<N; ++i){            // 미로 입력

        string row;                     // 행 입력
        cin >> row;

        for (int j=0; j<M; ++j){        // 행 별 좌표값 저장
            maze[i][j] = row[j]-'0';    // 행 별 좌표값들은 문자 형태이기 때문에, 숫자로 변환
        }
    }

    bfs(0, 0);                          // 미로 탐색 시작

    cout << dist[N-1][M-1];             // 도착 좌표까지의 거리 출력
}
```
다른 사람이 푼 풀이 역시 제 풀이와 로직이 모두 동일하여 부가적인 설명은 생략하도록 하겠습니다. 

## 최적화한 나의 풀이
최적화한 풀이가 다른 사람이 푼 풀이와 동일하여서 이하 설명은 생략하도록 하겠습니다.

## References
> https://wooono.tistory.com/410   