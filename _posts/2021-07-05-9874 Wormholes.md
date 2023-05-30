---
title: 9874 Wormholes
author: 신성일
date: 2021-07-05 19:10:44 +0900
categories: [알고리즘, beackjoon]
tags: []
---

## 알고리즘

- 페어링 쌍을 만들고, 사이클이 있는지 확인하는 작업을 모든 페어링 쌍을 대상으로 하면 됨.
- 헤멘 부분
  1. 사이클을 찾을때 같은 페어링쌍을 다시 방문하는게 아니라, 같은 점을 다시 방문하는지 확인해야함.
  2. check 를 언제 초기화해야하는지 헷갈림 -> 확실히하려면 매 시작점마다하면 됨.

## 여담

- 출척 Olympiad> USA Computing Olympiad 인데, 요즘 이 출처의 문제들을 풀면서 느낀 게, 이 출처의 문제들은 내용자체는 간단한데, 예외 또는 특수한 경우 처리가 매우 어렵다.
- 그리고 내겐 한번 '틀렸습니다.'가 뜨면 기존 알고리즘에 집착해서 새로운 생각을 못하는 단점이 있는거 같다.
- 앞으로는 문제풀때 배열 범위문제는 일어나지 않도록 크게 두고, 틀렸습니다가 뜨면 기존 알고리즘 재점검하고, 문제 없을경우 지체없이 특수한 경우를 생각해봐야겠다.

## 코드

```js
#include <iostream>
#include <cstring>
#define INF 2000000000

using namespace std;

int n, holes[15][2], ans, pairing[7][2];
bool visit[15], check[8][2];

void makePair(int cur, int deep);
bool findCycle(int s, int o);

int main() {
	cin >> n;
	for (int i = 0; i < n; i++)
		cin >> holes[i][0] >> holes[i][1];

	visit[0] = true;
	makePair(0, 0);

	cout << ans;
}

void makePair(int cur, int deep) {
	if (deep == n / 2) {
		for (int i = 0; i < 2; i++) {
			for (int j = 0; j < n / 2; j++)
				if (findCycle(j, i)) {
					ans++;
					return;
				}
		}
		return;
	}
	pairing[deep][0] = cur;
	for (int i = cur + 1; i < n; i++) {
		if (!visit[i]) {
			visit[i] = true;
			pairing[deep][1] = i;
			int j = 0;
			for (; j < n; j++)
				if (!visit[j])
					break;
			visit[j] = true;
			makePair(j, deep + 1);
			visit[i] = false;
			visit[j] = false;
		}
	}
}


bool findCycle(int s, int o) {
	int next = pairing[s][!o];
	check[s][o] = true;

	while (next != -1) {
		int cur = next;
		next = -1;

		int cr = holes[cur][1], cc = holes[cur][0], ni, nj, close = INF;
		for (int i = 0; i < n / 2; i++) {
			for (int j = 0; j < 2; j++) {
				if (cr == holes[pairing[i][j]][1] && cc < holes[pairing[i][j]][0] && holes[pairing[i][j]][0] < close) {
					close = holes[pairing[i][j]][0];
					ni = i;
					nj = j;
				}
			}
		}

		if (close != INF) {
			if (check[ni][nj]) {
				memset(check, 0, sizeof(check));
				return true;
			}
			check[ni][nj] = true;
			next = pairing[ni][!nj];
		}
	}
	memset(check, 0, sizeof(check));
	return false;
}
```
