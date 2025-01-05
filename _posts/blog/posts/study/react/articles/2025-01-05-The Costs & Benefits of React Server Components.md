https://www.youtube.com/watch?v=TJOiXyVKExg&ab_channel=RealWorldReact

### Summary

🎙️ In this talk, Jeff discusses the different architectures of using React in various applications, including client-only, server-side rendering (SSR), and React Server Components (RSC). He explains the trade-offs and complexities of each approach, highlighting how RSC introduces a new rendering environment and challenges in nesting server-only and client/server-rendered content.

### Facts

- React can be used in multiple ways, each with unique trade-offs.
- Jeff introduces React Server Components (RSC) and discusses their place in the React architecture spectrum.
- Client-only React is simple but lacks SEO and initial load speed.
- Server-side rendering (SSR) improves SEO and initial load speed but increases complexity.
- React Server Components (RSC) offer better performance for complex apps but introduce increased complexity.
- RSC introduces a new rendering environment: server-only, which coexists with client and server/client-rendered content.
- Nesting client and server components within each other in RSC can lead to unexpected complexities.
- RSC implementation was challenging and required close collaboration with the React team.
- RSC's benefits shine in large, complex apps operating at high scale.


### 노트
- 마지막 예시
	- fetch 함수가 다르면 다른 캐시에 저장됨.
	- 같은 endpoint로 요청하더라도, 다른 fetch 함수로, 다른 캐싱 설정을 주어 요청하면 다르게 동작한다.
- 서버컴포넌트를 사용하면 복잡도가 올라가니, 서버컴포넌트가 굳이 필요없다면 사용하지 않는 것이 좋음