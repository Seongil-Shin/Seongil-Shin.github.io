---
title: 14529 Where's Bessie
author: 신성일
date: 2021-07-08 21:10:44 +0900
categories: [알고리즘, beackjoon]
tags: []
---

## 알고리즘

- 그냥 조건에 맞춰서 풀면 되는 문제이다.
- 가능한 모든 사각형에 대해 검사하고, pcl을 뽑아낸 다음에, 마지막으로 '다른 pcl에 포함되는 pcl'을 제거하고 출력하면 된다.
- '한 색은 연속된 지역이 하나이고, 다른 색은 두개 이상이어야한다. ' 라는 조건은 다음과 같이 하면된다.
  - 인접한 같은 색에 대해서만 BFS를 진행하고, BFS 진행된 총 수 t를 센다.
  - t가 현재 지역에서 그 색의 수와 같으면 첫번째 조건이 만족되었다고 표시, 그것이 아니라면 두번째 조건이 만족했다고 표시하는데, 두번째 조건은 만족할때마다 1을 더해주는 카운터로 표현하자.
  - 위와같이 하면
    - '연속된 지역이 하나인 색'이 없을 경우 첫번째 조건이 만족하지 않음
    - 두 색 모두 하나로 연결되었을때는 두번째 조건이 만족되지 않음.
    - 따라서 정답일때만 pcl에 추가해줄 수 있다.

## 코드

```c++
#include <iostream>
#include <vector>
#include <cstring>

using namespace std;

char map[22][22];
int n, colorCheck[26], os[4][2] = { {-1,0}, {0,1}, {1,0},{0,-1} };
vector<pair<pair<int, int>, pair<int, int>>> pcl;
bool visit[22][22];
pair<int, int> q[420];

void getPCL();
void checkSingleRectagle(int, int,int,int);

int main() {
	cin >> n;
	for (int i = 0; i < n; i++)
		for (int j = 0; j < n; j++)
			cin >> map[i][j];
	getPCL();

	int last = pcl.size() - 1;
	for (int i = last; i >=0; i--) {
		int si = pcl[i].first.first, sj = pcl[i].first.second, w = pcl[i].second.first, h = pcl[i].second.second;
		for (int j = 0; j < i; j++)
			if (si >= pcl[j].first.first && sj >= pcl[j].first.second && w <= pcl[j].second.first && h <= pcl[j].second.second) {
				pcl.pop_back();
				break;
			}
	}

	cout << pcl.size() << endl;
}

void getPCL() {
	for (int h = n - 1; h >= 0; h--) {
		for (int w = n - 1; w >= 0; w--) {
			for (int i = 0; i < n - w; i++) {
				for (int j = 0; j < n - h; j++) {
					checkSingleRectagle(i, j, i + w, j + h);
					memset(colorCheck, 0, sizeof(colorCheck));
				}
			}
		}
	}
}

void checkSingleRectagle(int si, int sj, int w, int h) {
	for (int i = 0; i < pcl.size(); i++)
		if (si >= pcl[i].first.first && sj >= pcl[i].first.second && w <= pcl[i].second.first && h <= pcl[i].second.second)
			return;

	int count = 0;
	for (int i = sj; i <= h; i++) {
		for (int j = si; j <= w; j++) {
			if (!colorCheck[map[i][j] - 'A'])
				count++;
			colorCheck[map[i][j] - 'A']++;
			if (count > 2)
				return;
		}
	}
	if (count != 2)
		return;


	bool check1 = false;
	int check2 = 0;
	for (int i = sj; i <= h; i++) {
		for (int j = si; j <= w; j++) {
			if (!visit[i][j]) {
				char c = map[i][j];
				int temp = colorCheck[c - 'A'], curIdx = 0, lastIdx = 0;
				visit[i][j] = true;
				q[lastIdx++] = { i,j };
				while (curIdx != lastIdx) {
					int ch = q[curIdx].first, cc = q[curIdx].second;
					curIdx++;
					temp--;

					for (int k = 0; k < 4; k++) {
						int nh = ch + os[k][0], nc = cc + os[k][1];
						if (nh >= sj && nh <= h && nc >= si && nc <= w && !visit[nh][nc] && map[nh][nc] == c) {
							q[lastIdx++] = { nh, nc };
							visit[nh][nc] = true;
						}
					}
				}
				if (temp == 0)
					check1 = true;
				else
					check2++;
			}
			if (check1 && check2 >= 2)
				break;
		}
	}
	memset(visit, 0, sizeof(visit));

	if (check1 && check2 >= 2)
		pcl.push_back({ {si, sj}, { w, h} });
}
```
