---
title: IPv6
author: 신성일
date: 2022-02-30 14:00:00 +0900
categories: [study, network]
tags: [network]
---

## **IPv6**

- 128비트 사용.

- Multicast 대신에 Broadcast를 함.

- ICMPv6

  - ARP, IGMP 기능흡수
    - ARP : IP주소를 MAC 주소로 변환하는 프로토콜

- IPv4 에서 IPv6로의 전환 정책

  - Dual stack : 두가지 다 지원하도록 하여, 두가지 주소를 모두 사용할 수 있도록 하는 것
  - tunneling : 종단 사이에 IPv4를 쓰는 홉이 있을때, 그 중간 단계에서만 IPv6를 터널링으로 IPv4로 감싸 통신
  - header translation : 종단에서만 IPv4를 사용하면, IPv6를 해석하여 IPv4로 변경한 다음에 전달
