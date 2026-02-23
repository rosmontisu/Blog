---
title: '[Algorithm] Network Flow'
published: 2026-02-04
description: '네트워크 유량: Network Flow & Max-Flow Min-Cut(MFMC) 정리 및 BFS기반의 Edmonds-Karp 알고리즘을 사용해 최대 유량 구하기'
image: ''
tags: [Algorithm, Graph, Network Flow]
category: 'Algorithm'
draft: false 
lang: ''
---

# 조건
정점 u -> v 일때
- 간선의 용량 : `c(u, v)`
- 실제 흐르는 유량 : `f(u, v)`
- 전체 정점의 집합 : `V`

다음 3가지 속성을 만족해야한다.
## 1. 용량 제한 속성
- `f(u, v)` <= `c(u, v)`
	- 자명한 속성
	- 그래프의 한 간선에서 흐르는 유량은 용량을 초과할 수 없다.

## 2. 유량의 대칭성
- `f(u, v)` == `-f(v, u)`
	- u -> v 로 양수의 유량을 보낸다는 것은, v -> u 로 음수의 유량을 보내는 것으로 간주
	- 역방향 간선에 음의 유량이 흐르는 것으로 처리(잔여 그래프; Residual Graph생성)하여, 나중에 경로를 취소할 수 있음
## 3. 유량의 보존
- `∑f(in) == ∑f(out)` (들어온 양 = 나간 양)
	- 소스와 싱크를 제외한 모든 정점에는(파이프의 이음새)에는 물이 고일 수 없음
## 소스&싱크 개념

소스는 유량이 시작되는 정점이고, 싱크는 유량이 끝나는 정점이기 때문에, 이 두가지 정점에 한해서 유량의 보존 성질이 성립하지 않는다.
- 소스 (source) : 수원지
- 싱크 (sink) : 목적지

유량 그래프에서 `소스` -> `싱크`로 최대로 흐를 수 있는 유량을 구하는 문제를 `네트워크 유량` 문제라고 한다.

---

# 포드-폴커슨 방법 (Ford-Fulkerson method)
## 동작 원리
- u -> v 로 보낼 수 있는 잔여 용량을 `r(u, v)`라고 할때
	- `r(u, v)` == `c(u, v) - f(u, v)`
- 일단 임의의 경로로 용량을 흘려보내고
- 기존에 선택했던 경로를 취소하고, 새로운 경로로 더 개선된 답을 찾는다.
## 최대 유량 최소 컷 정리 (Max-Flow Min-Cut)

### 컷 (cut)
- 그래프를 두개의 조각으로 토막낸 것
- 유량 그래프에서의 컷은, 시작점인 소스와 종점인 싱크가 다른 집합에 속하도록 그래프를 쪼갠 것
	- 소스가 있는 컷 : S
	- 싱크가 포함된 컷 : T
- S -> T 로 가는 간선들의 용량 합 : 컷 S, T의 용량
- S -> T 로 가는 실제 유량 : 컷 S, T의 유량
	- (중요) 컷의 유량은 `역방향 간선`의 -유량도 더해야 함

### 속성
- 컷의 유량은 `소스 -> 싱크`로 가는 총 유량과 같다.
- 컷의 유량은 **컷의 용량(capacity)보다 크지 않다**.

### 추론
- (유량 보존) 임의의 컷을 통과하는 순 유량은 모두 동일하며, 이는 `소스 -> 싱크`로 흐르는 총 유량과 같다.
- (컷 용량 제한) 컷을 통과하는 유량은 물리적으로 그 컷의 용량을 초과할 수 없다. `유량 <= 컷의 용량`
- (상한) 따라서 그래프의 모든 컷 중 **가장 작은 용량을 가진 컷(최소 컷)** 의 용량이 전체 유량의 한계치가 된다.
- 만약 `현재 유량 == 컷의 용량`이 되는 상태(모든 간선이 꽉 찬 컷)를 하나라도 발견하면, 그 이상 유량을 늘리는 것은 불가능하므로 이때가 `최대 유량`이다.
	- 더 이상 증가 X -> 컷의 용량 포화

### 결론
- `최대 유량` == `최소 컷의 용량`
- 즉, `Max-Flow` == `Min-Cut` 정리가 성립한다.

---
# 애드몬드-카프 알고리즘 (Edmonds-Karp)
## 이론
- BFS로 시작점에서 가장 간선 개수가 적은 경로(최단 경로)부터 찾는다.
	- 이러면 유량을 흘려보내는 횟수가 O(VE)로 제한된다.
## 알고리즘 흐름

아래의 과정을 경로가 없을 때까지 무한 반복한다.
### 1. 길 찾기 (BFS)
- 소스(S) -> 싱크(T) 로 가는 경로를 BFS로 탐색
	- 이동 조건: `현재 유량 < 용량` 인 간선만 이동 가능 (물이 꽉 차면 이동 X)
	- 이때, `parent`배열로 경로 역추적 필요
### 2. 병목 지점 찾기 (Bottleneck)
- 싱크(T)에 도착했다면, `parent`배열을 타고 소스(S)까지 올라간다
- 이 경로상에 파이프 중 `잔여 용량(Capacity - Flow)`이 가장 **작은 값**을 찾는다.
- `잔여 용량(amount)` 이 값이 이번 턴에 흘려보낼 수 있는 유량 (가장 좁은 파이프만큼만 물이 흐르므로)

### 3. 유량 흘리기 (Update & Reverse Edge)
- 다시 경로를 거슬러 올라가면서 유량 데이터를 갱신
- 정방향 : `flow[u][v] += amount` (물 채우기)
- 역방향 : `flow[v][u] -= amount` (반대 유량은 감소 -> 음수 유량)

## 예제
### 구조
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>
using namespace std;

const int INF = 0x3f3f3f3f;
const int MAX = 1000;

// c: Capacity 용량
// f: Flow 현재 유량
int c[MAX][MAX]; // cur -> nxt
int f[MAX][MAX]; // cur -> nxt
vector<int> adj[MAX];

// 최대 유량 구하기 S(source) -> T(sink; target)
int getMaxFlow(int source, int sink)
{
    int totalFlow = 0;
    while (true)
    {
        // 1. 길 찾기 (BFS)
        vector<int> parent(MAX, -1);
        queue<int> q;
        q.push(source); // 시작점
        parent[source] = source; // 방문 표시

        while (!q.empty())
        {
            int cur = q.front(); q.pop();
            for (int nxt : adj[cur])
            {
                if ((c[cur][nxt] - f[cur][nxt]) <= 0) continue; // 잔여 용량이 남지 않음
                if (parent[nxt] != -1) continue; // 이미 방문함

                q.push(nxt);
                parent[nxt] = cur; // cur -> nxt 경로 기록
                if (nxt == sink) break; // 싱크 도착
            }
        }

        // 2. 싱크에 도달 못함. 경로가 없음
        if (parent[sink] == -1) break;

        // 3. 병목 지점 찾기 (Bottleneck) - 가장 작은 파이프
        int amount = INF;
        for (int p = sink; p != source; p= parent[p])
        {
	        int prv = parent[p];
	        amount = min(amount, c[prv][p] - f[prv][p]);
        }

        // 경로 역추적
        for (int p = sink; p != source; p = parent[p])
        {
            int prv = parent[p];
            f[prv][p] += amount; // u->v 정방향
            f[p][prv] -= amount; // v->u 역방향
        }
        
        totalFlow += amount;
    }
    
    return totalFlow;
}
```

### 연습 문제 - BOJ 6086 최대 유량 구현
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>
using namespace std;

const int INF = 0x3f3f3f3f;

int c[150][150]; // capacity
int f[150][150]; // flow 0
vector<int> adj[150]; // 파이프 연결 list

int main()
{
    int n; cin >> n;
    for (int i = 0; i < n; i++)
    {
        char u, v;
        int w;
        cin >> u >> v >> w;
        
        // 양방향 그래프
        c[u][v] += w;
        c[v][u] += w;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    int ans = 0;
    while (true)
    {
        // S(source) -> T(target; sink)
        int s = 'A';
        int t = 'Z';

        vector<int> p(150, -1); // BFS 기록용
        queue<int> q;
        q.push(s);
        p[s] = s; // 시작점 부모

        // BFS로 가능한 파이프 경로 찾기
        while (!q.empty())
        {
            int cur = q.front(); q.pop();
            for (int nxt : adj[cur])
            {
                if (p[nxt] != -1) continue; // 이미 방문
                if (c[cur][nxt] - f[cur][nxt] <= 0) continue; // 이미 다씀

                q.push(nxt);
                p[nxt] = cur; 
                if (nxt == t) break; // sink 도착
            }
        }

        // BFS로 sink에 도달 못함. (최대 유량임)
        if (p[t] == -1) break;

        // 거슬러 올라가면서 최소 찾기
        int amount = INF;
        for (int cur = t; cur != s; cur = p[cur])
            amount = min(amount, (c[p[cur]][cur] - f[p[cur]][cur]));
        for (int cur = t; cur != s; cur = p[cur])
        {
            f[p[cur]][cur] += amount; // 정방향 : 유량 늘림
            f[cur][p[cur]] -= amount; // 역방향 : 유량 줄임 (다시 빼는경우)
        }
        ans += amount;
    }

    cout << ans;
    return 0;
}

```
