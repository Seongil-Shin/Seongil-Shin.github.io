---
title: 2024-08-31-Esbuild to reduce build times by 90%
author: 신성일
date: 2024-08-31 19:45:24 +0900
categories: study, article
tags:
  - "#study"
---
[원본](https://blog.1password.com/new-extension-build-system/)

기존에 빌드가 오래 걸렸던 원인들
- 웹팩, 롤업 작업 시간이 대부분이었음
- 작은 의존성들이 하나씩 빌드됨. 이것들은 동시에 진행될 여지가 있음
- 몇몇 의존성은 필요한 것보다 오래 빌드됨. 
- 변경되지 않은 파일에 대해서 typeshare 를 재실행하지 않기위해 find를 실행했으나, 그냥 typeshare를 실행하는 것보다 find 실행시간이 더 길었음

해커톤을 통해 몇몇 시도한 해결책
- esbuild를 webpack, rollup의 로더로 사용하여 성능 개선을 이루어냄.
	- rollup 에서는 80%의 성능개선을 이루어냄
- esbuild로 webpack, rollup을 완전히 대체함 -> 90%의 성능개선

실제 서비스 적용
- esbuild, with typechecking
	- `tsc`가 너무 느린 문제를 해결하기위해, `esbuild-plugin-typecheck`를 사용함. 
	- 이를 통해 논블록킹으로 tsc를 실행가능하게 한다
	- 여기에 더해 에러메세지를 포맷팅하여 가독성을 높였고, 모든 빌드 입력 파일이 제대로 타입 체킹되었는지 확인하는 로직을 추가함
- production bundle size improvements
	- esbuild의 [bundle size analyzer](https://esbuild.github.io/analyze/)를 사용하여 불필요하게 번들 사이즈를 늘리는 원인을 찾아내어 크기를 줄일 수 있었다
