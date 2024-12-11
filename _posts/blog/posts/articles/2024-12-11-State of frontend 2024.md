---
title: State of frontend 2024
author: 신성일
date: 2024-12-10 19:00:00 +0900
categories: study, article
tags:
  - "#study"
---
https://tsh.io/state-of-frontend/#data

## Chapter 3. Teams & Technology

### 02. Frameworks

- svelte 는 순풍중인거 같고 vue.js 는 생각보다 잘나가는듯
- 프레임워크는 Astro, Nuxt, Sveltekit, Remix 가 배울만 한거 같고, astro 가 생각보다 잘나가는듯

### 03. Libraries

- validation : zod 배워보자
- date management : date-fns, Day.js, moment 많이 씀. moment는 성능상 단점이 있어 비선호도가 많음
- state management : React context 를 가장 많이 씀. 그 외로는 zustand가 눈에띔

### 04. Data

- axios, fetch, TanStack Query 등 일반적으로 많이 쓰는게 순위가 높음
- 새로운 툴로는 tRPC 가 유망한듯
### 05. Hosting

- Vercel이 AWS를 이김

### 07. Micro frontend

- micro-frontend 자체를 많이 안쓰는 추세 인듯
- 지식만 갖고 있어보자

### 08. Package manager

- npm 이 제일 많고, yarn/pnpm 이 동등
- 런타임으로 bun 을 사용하는 경우가 많아져서 bun도 많이 쓰는듯 (자체 패키지매니저 기능 탑재)

### 09. Javascript runtime

- Node가 지배적이고 그 다음이 Bun
- 일시적일 수 있으나 Bun이 생각보다 높은 수치를 보임
- Deno는 나중에 오를수 있으나 현재는 Bun 보다도 현저히 낮은 수치



## Chapter 4. Developer & User Experience

### 01. Typescript

- 대부분 사용중이고, 앞으로도 많이 사용할거라는 전망
- JS를 대체할거라는 인식이 더 강해짐

### 03. Progressive Web Apps

- 현재 인기도 유지 ~ 약간 상승 정도로 보는듯

### 04. Design system

- shadcn/ui (with tailwind) 가 가장 많이 사용되고, 그 다음은 Material UI

### 05. Styling tools

- Plain CSS > SCSS > Tailwind CSS > CSS Modules > Styled Component

### 06. Testing

- 80% 가까운 사람들이 테스트를 수행한 적이 있음.
- 유닛테스트를 제일 많이 하고, End-to-End UI test랑 Integration test 도 꽤 하는듯 
- 테스팅 툴로는 Jest > Cypress > Vitest > Testing Library > Playwright 순으로 많이 사용함


### 07. Code management

- Cursor 써볼만 한듯
- VSCode 에 copilot 이랑 연계해서 강화했다는데 한번 찾아볼 가치 있어보임

### 09. Building tools

- esbuild, vite 조사 + 사용해보기 

