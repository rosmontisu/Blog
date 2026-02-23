---
title: '[Algorithm] Permutation Cycle Decomposition'
published: 2026-02-11
description: '순열 사이클 분할 개념, DFS를 이용해 배열 정렬 최소 교환 횟수를 구하는 방법을 알아보자.'
image: ''
tags: [Algorithm, Graph, Permutation Cycle]
category: 'Algorithm'
draft: false 
lang: 'ko'
---

# 개념
알고리즘 문제에서 배열을 정렬하기 위해 필요한 `최소 교환 횟수`를 묻는다면, 높은 확률로 순열 사이클 분할 문제이다.


- $N$개의 원소로 이루어진 순열
- 현재 인덱스($i$)와 그 위치에 있는 값($A[i]$)을 간선으로 연결하면($i \rightarrow A[i]$)
- 배열은 서로 겹치지 않는 여러 개의 사이클(Cycle)로 분해된다.

이것을 순열 사이클 분할(Permutation Cycle Decomposition)이라고 한다.

## 예시


아래와 같은 배열이 있다고 가졍해보자. 

$$N = 5, \quad Arr = \{ 2, 4, 5, 1, 3 \}$$   


| 1-based index ($i$)  |  1  |  2  |  3  |  4  |  5  |
| :-: | :-: | :-: | :-: | :-: | :-: |
| 값 ($A[i]$) |  2  |  4  |  5  |  1  |  3  |

이 관계를 그래프로 따라가 보면 2개의 독립적인 사이클이 만들어진다.
- **Cycle 1**: $1 \rightarrow 2 \rightarrow 4 \rightarrow 1$ (길이 3)
- **Cycle 2**: $3 \rightarrow 5 \rightarrow 3$ (길이 2)

## 최소 교환 횟수 공식

순열 사이클 분할을 이용하면 배열을 정렬하는 데 필요한 **최소 교환 횟수**를 구할 수 있다.

$$\text{최소 교환 횟수} = N - (\text{사이클의 개수})$$

- 왜 이런 공식이 나올까?

1. **길이가 1인 사이클**: 이미 제자리에 있는 원소이므로, 교환이 필요 없다. (0회)
    
2. **길이가 $K$인 사이클**: $K$개의 원소가 서로 자리를 바꾸며 얽혀 있다. 이를 모두 제자리로 보내려면 **$K-1$번의 교환**이 필요하다.
    

전체 $N$개의 노드가 $C$개의 사이클로 분할되어 있고, 각 사이클의 길이를 $L_1, L_2, \dots, L_C$라고 할때, 전체 교환 횟수는 각 사이클을 정렬하는 횟수의 합이다.

$$\begin{aligned} \text{Total Swaps} &= \sum_{i=1}^{C} (L_i - 1) \\ &= \sum_{i=1}^{C} L_i - \sum_{i=1}^{C} 1 \\ &= N - C \end{aligned}$$

위의 예시 ${ 2, 4, 5, 1, 3 }$에 적용해 보면:
- 전체 노드 개수 $N$ = 5
- 사이클의 개수 $C$ = 2
- 최소 교환 횟수 = $5 - 2 = 3$번
# 예제
## 메인 로직
DFS나 BFS로 미방문 노드 사이클을 탐색하고, 그 횟수를 카운팅한다.
```cpp
#include <iostream>
#include <vector>
using namespace std;
const int INF = 0x3f3f3f3f;

int n;
int arr[INF];
bool vis[INF];

int main()
{
    int cnt = 0;

    for (int i = 1; i <= n; i++)
    {
        // // 미 방문 노드 발견 (새로운 사이클 발견)
        if (!vis[i])
        {
            cnt++;
            int cur = i;

            // DFS
            while (!vis[cur])
            {
                vis[cur] = true;
                cur = arr[cur]; // 다음 노드로 이동
            }
        }
    }

    // n - 사이클 개수
    cout << n - cnt;
}
```



## 연습 문제 - BOJ 10451 순열 사이클
순열 사이클의 수를 출력하는 문제다.

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main()
{
    int t; cin >> t;
    while (t--)
    {
        int n; cin >> n;
        vector<int> vec(n + 1);
        for (int i = 1; i <= n; i++)
            cin >> vec[i];

        int cnt = 0;
        vector<bool> vis(n + 1, false);
        for (int i = 1; i <= n; i++)
        {
            // 사이클 찾았음
            if (vis[i] == false)
            {
                cnt += 1;
                int cur = i;

                // DFS
                while (vis[cur] == false)
                {
                    vis[cur] = true;
                    cur = vec[cur];
                }
            }
        }
        cout << cnt << '\n';
    }
}
```

## 연습 문제 - BOJ 34078 벨과 와이즈의 가게 홍보
내림차순 케이스를 조심하자..
```cpp
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

int main()
{
    int n; cin >> n;
    vector<int> vec(n + 1, 0);
    for (int i = 1; i <= n; i++)
        cin >> vec[i];

    // 웅나 수 = n탐색 돌리기
    // 웅나 시간 = n - 사이클의 수

    int cnt1 = 0;
    vector<bool> vis(n + 1, false);
    for (int i = 1; i <= n; i++)
    {
        if (vis[i] == false)
        {
            int cur = i;
            cnt1 += 1; // 사이클 시작점 발 견
            while (vis[cur] == false)
            {
                vis[cur] = true;
                cur = vec[cur];
            }
        }
    }

    // 내림차순 검사 { 4 3 2 1 } 케이스
    int cnt2 = 0;
    vis.assign(n + 1, false);
    for (int i = 1; i <= n; i++)
    {
        if (vis[i] == false)
        {
            int cur = i;
            cnt2 += 1; // 사이클 시작점 발 견
            while (vis[cur] == false)
            {
                vis[cur] = true;
                cur = vec[n - cur + 1];
            }
        }
    }
	
	// 모든 웅나의 키는 다르므로, n-2
    cout << n - 2 << ' ' << n - max(cnt1, cnt2);

}
```