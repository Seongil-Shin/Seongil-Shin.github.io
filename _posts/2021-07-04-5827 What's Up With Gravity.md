---
title: 5827 What's Up With Gravity
author: 신성일
date: 2021-07-04 21:10:44 +0900
categories: [알고리즘, beackjoon]
tags: []
---

## 알고리즘

- BFS 사용

1. 시작할때 C는 떨구고 시작.
2. bfs를 진행하는데 왼쪽, 오른쪽, 중력 바꿔서 검사하고 떨군다.
   1. 떨어지는 도중에 D를 만나면, 만났다고 표시 -> 끝나고 바로 ans 업데이트
3. 이때 visit은 해당 위치에서 해당 중력일때 최소 flip을 저장함
   1. 나는 이동후 떨구기 전과 떨군 후 두가지만 visit 표시했지만, 더 할 수도 있을 거 같음.

- 시간 더 줄이기
  - 우선순위 큐 사용하여, flip 횟수가 적은 것을 우선하기
  - visit 체크를 더 할 수 있는 부분은 더하기

## 코드

```c++
#include <iostream>
#include <list>
#define INF 2000000000

using namespace std;

int n, m, os[2] = { 1, -1 }, visit[500][500][2], ans = INF;
char map[500][500];
pair<int, int> c;
list<pair<pair<int, int>, pair<bool, int>>> l;
bool meetD;

bool fall(bool g, int* r, int* c);
void bfs();
void afterCheck(int *r, int *c, bool g, int f);
int min(int a, int b) {
	return a < b ? a : b;
}

int main() {
	cin >> n >> m;
	for (int i = 0; i < n; i++) {
		string s;
		cin >> s;
		for (int j = 0; j < m; j++) {
			map[i][j] = s[j];
			if (s[j] == 'C')
				c = { i, j };
			visit[i][j][0] = visit[i][j][1] = INF;
		}
	}

	if (!fall(0, &c.first, &c.second)) {
		cout << -1;
		return 0;
	}

	bfs();
	if (ans != INF)
		cout << ans;
	else
		cout << -1;
}

bool fall(bool g, int* r, int* c) {
	while (((g == 0 && *r < n - 1) || (g == 1 && *r > 0)) && map[*r + os[g]][*c] != '#') {
		if (map[*r][*c] == 'D')
			meetD = true;
		*r += os[g];
	}

	if (map[*r][*c] == 'D')
		meetD = true;

	if ((g == 0 && *r == n - 1) || (g == 1 && *r == 0))
		return false;
	return true;
}

void bfs() {
	l.push_back({ {c.first, c.second}, {0,0} });
	visit[c.first][c.second][0] = 0;

	while (!l.empty()) {
		int cr = l.front().first.first, cc = l.front().first.second, g = l.front().second.first, f = l.front().second.second;
		l.pop_front();

		if (map[cr][cc] == 'D') {
			ans = min(ans, f);
			continue;
		}

		for (int i = 0; i < 2; i++) {
			int nr = cr, nc = cc + os[i];
			if (nc >= 0 && nc < m && f < visit[nr][nc][g] && map[nr][nc] != '#')
				afterCheck(&nr, &nc, g, f);
		}

		if (++f < visit[cr][cc][!g])
			afterCheck(&cr, &cc, !g, f);
	}
}

void afterCheck(int* r, int* c, bool g, int f) {
	visit[*r][*c][g] = f;
	bool temp = fall(g, r, c);

	if (meetD) {
		meetD = false;
		ans = min(ans, f);
	}
	else if (temp) {
		visit[*r][*c][g] = f;
		l.push_back({ {*r,*c}, {g, f} });
	}
}
```
