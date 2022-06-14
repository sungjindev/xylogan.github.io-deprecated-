---
title: 고득점 Kit_ 해시 - Level1 - 완주하지 못한 선수
categories: [Computer science, Algorithm]
tags: [Hash, Level1, Programmers, 고득점, 알고리즘, 완주하지 못한 선수, 코딩 테스트, 프로그래머스, 해시]
---

**!본 포스팅은 프로그래머스 코딩테스트 고득점 Kit - [완주하지 못한 선수](https://programmers.co.kr/learn/courses/30/lessons/42576) 풀이입니다.**

# 처음으로 맞춘 풀이
``` cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

string solution(vector<string> participant, vector<string> completion)
{

  sort(participant.begin(), participant.end(), [](string a, string b) //시간 복잡도 문제 때문에 각각을 정렬한 뒤에 비교하는 방식을 취함
       { return a < b; });
  sort(completion.begin(), completion.end(), [](string a, string b)
       { return a < b; });
  for (int i = 0; i < completion.size(); i++)
  {
    if (completion[i] != participant[i])
    {
      string answer = participant[i];
      return answer;
    }
  }
  string answer = participant.back();
  return answer;
}
```
- 이 문제가 코딩테스트 준비의 첫걸음이라고 말할 수 있을 정도로, 코딩테스트 공부를 본격적으로 처음 공부해본 저는, 처음에는 이중 포문을 사용해서 문제 풀이를 시작했었습니다. 하지만, 시간 복잡도에 의해서 효율성 측면에서 어김없이 오답...

- 그래서 구글링을 통해 정렬을 한 뒤 비교를 하는 방식을 태하여 문제 풀이를 이어 나갔습니다.

` 
Tip! 문제 속 입력 값의 개수를 보고 시간 복잡도를 많이 따지는 문제인지 아닌지를 판가름하여 해결하면 더욱 편리하게 풀 수 있습니다.
`


# 다른 사람의 풀이
``` cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

string solution(vector<string> participant, vector<string> completion)
{
  string answer = "";
  sort(participant.begin(), participant.end());
  sort(completion.begin(), completion.end());
  for (int i = 0; i < completion.size(); i++)
  {
    if (participant[i] != completion[i])
      return participant[i];
  }
  return participant[participant.size() - 1];
  //return answer;
}
```
- 다른 사람의 풀이 역시 저와 일치하는 풀이법이네요. 정렬을 한 뒤에 비교를 해준 풀이 방식입니다.

# 최적화한 나의 풀이
- 최적화된 풀이가 처음에 본인이 푼 풀이와 거의 비슷하여 생략합니다. 



