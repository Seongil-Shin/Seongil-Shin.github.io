---
title: 10021 watering the field
author: 신성일
date: 2021-06-24 19:38:44 +0900
categories: [알고리즘, beackjoon]
tags: []
---

## 알고리즘

- 크루스칼
- 모든 필드 사이의 거리를 계산하고, 양 쪽 필드의 번호와 거리를 벡터에 저장한다.
- 거리순으로 정렬하고, 크루스칼 알고리즘 시행
- 이때, 단순히 union-find 알고리즘으로 하면 find에서 시간초과가 난다.
- 따라서 set을 활용해서 크루스칼을 하면 된다.

## 헤맨 지점

1. set을 이용해서 할때, 서로 다른 집합이었다가 합쳐질때, 하나만 옮기면 안된다. 그 set에 속해있는 모든 점을 다 옮겨야함
2. 꼭 첫번째 집합으로 합쳐지지 않는다. 따라서, 모든 점들이 같은 집합에 있는지 하나하나 확인해봐야한다.

## 코드

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <set>
using namespace std;

struct node {
	int first;
	int second;
	int length;

	bool operator< (node a) {
		return length < a.length;
	}
};

vector<pair<int, int>> field;
vector<node> lengths;
vector<set<int>> unionset;
int N, C, parent[2000];
long long result;

void calcLengths();
bool kruskal();
int power(int a) {
	return a * a;
}

int main() {
	cin >> N >> C;
	int a, b;
	while (N--) {
		cin >> a >> b;
		field.push_back({ a, b });
	}
	N = field.size();

	calcLengths();
	memset(parent, -1, sizeof(parent));
	sort(lengths.begin(), lengths.end());

	if (kruskal())
		cout << result;
	else
		cout << "-1";
}

void calcLengths() {
	for (int i = 0; i < N; i++)
		for (int j = i + 1; j < N; j++)
			lengths.push_back({ i, j, power((field[i].first - field[j].first)) + power((field[i].second - field[j].second)) });
}

bool kruskal() {
	for (int i = 0; i < lengths.size(); i++) {
		int f = lengths[i].first, s = lengths[i].second, l = lengths[i].length;
		int sp = parent[s], fp = parent[f];
		if (l < C)
			continue;

		if (fp == -1 && sp == -1) {
			set<int> t;
			t.insert(f);
			t.insert(s);
			unionset.push_back(t);
			parent[f] = unionset.size()-1;
			parent[s] = parent[f];
			result += l;
		}
		else if (fp == -1) {
			unionset[sp].insert(f);
			parent[f] = sp;
			result += l;
		}
		else if (sp == -1) {
			unionset[fp].insert(s);
			parent[s] = fp;
			result += l;
		}
		else if(fp != sp) {
			for (auto iter = unionset[sp].begin(); iter != unionset[sp].end(); iter++) {
				parent[*iter] = fp;
				unionset[fp].insert(*iter);
			}
				result += l;
			unionset[sp].clear();
		}
	}
	int root = parent[0];
	for (int i = 0; i < N; i++) {
		if (parent[i] != root)
			return false;
	}

	return true;
}
```
