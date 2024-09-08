---
title: template
author: 신성일
date: 2024-09-08
categories: study, article
tags:
  - "#study"
---
https://medium.com/@thecodingteacher_52591/why-nextjs-sucks-0352de93071b

저자가 next.js를 사용하면서 느낀 왜 next.js가 복잡한 프로젝트에서는 별로인지 설명한 글이다. 크게 네가지 이유를 들어 설명한다

- NodeJS sucks for SSR
	- nodeJS는 싱글 스레드이며 이는 I/O 처리를 하는데 좋지만, 복잡한 SSR을 함께 다루면 취약해질 수 있다.
	- 요청량이 많아지게 되면 페이지 렌더링 작업이 큐에 쌓이게 되고, 이는 점점 겉잡을 수 없게 되어 응답이 오지 않게 되는 경우가 발생할 수 있다는 것이다.
	- 따라서 next.js 에서는 Static page pregeneration 과 같은 기능을 제공하지만, 그냥 서버 렌더링을 하지 않는 것이 좋다는 게 저자의 생각이다.
- Security takes a back seat to coolness
	- Next.js 에는 suspense, appDir 기능, next/font 등 inline script를 요구하는 기술이 있다
	- 보통은 브라우저에서 inline script 를 막도록하기에 문제가 없으나, next.js의 inline script 기술을 사용하며 이를 허용하게 되고, 이는 XSS에 노출되도록 한다.
- NextJS doesn't respect your time
	- NextJS 의 API는 불안정하고, 메이저 업데이트를 6개월마다 한번씩하는데, 그 업데이트를 하는 것도 어렵게 만들어놨다
- The hosting infrastructure is much worse
	- SPA랑 CDN으로 호스팅하면 매우 적은 비용으로 많은 요청을 커버할 수 있다
	- Nextjs로 호스팅을 하게 되면 두가지 선택지가 있다
		- 서버리스로 호스팅하고, 간단한 DDOS만으로 많은 요금에 취약해지기
		- EC2에 띄우고 가용성을 잃기 (scale up이 오래 걸리기에)
	- 이는 많은 비용을 청구한다




개인적으로 `NodeJS sucks for SSR` 부분의 논지는 동의하지 않는데, next.js를 사용하면서 SSR 로 인한 성능 이슈는 겪은 적이 없기 때문이다. 정확한 측정을 해보진 않았지만 렌더링에는 시간이 오래 걸리지 않고, 지금까지 겪은 성능이슈는 렌더링에 필요한 데이터를 얻기 위해 API 요청하는 과정에서 발생했다. 

만약 SSR에 정말로 성능 이슈가 있다면 글에서도 언급된 오토스케일링을 통해 해결할 수 있다고 본다. (글에서는 이것도 부정적으로 봤지만)

일반적으로 API 요청에서 성능 이슈가 발생하는 패턴으로 미루어보아 SSR을 제거해봤자 성능 개선이 드라마틱하게 발생하지 않을 것 같고, 요청이 몰릴때를 대비해 SSR을 제거하는 것은, 이슈가 발생하지 않을 훨씬 긴 시간 동안 SSR의 유저 경험 개선 효과를 포기하는 것으로 보인다

`NextJS doesn't respect your time` 파트는 중립이다. 딱히 불안정하다고 느낀적은 없고 업데이트에 어려움을 겪지도 않았기 때문이다. 몇몇 특수한 기능을 사용하는 사람들에겐 그럴 수 있으니 중립의견을 잡았다.


`The hosting infrastructure is much worse` 파트도 서버 비용을 아끼기 위한 방책들이 있다고 코멘트에는 나와있다 (DDOS protection, spend limit, request throttling 등)