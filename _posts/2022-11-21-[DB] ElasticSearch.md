---
title: ElasticSearch
author: 신성일
date: 2022-11-21 21:18:26 +0900
categories: [study, database]
tags: [database, elastic, search]
---

> Elasticsearch는 시간이 갈수록 증가하는 문제를 처리하는 분산형 RESTful 검색 및 분석 엔진입니다. Elastic Stack의 핵심 제품인 Elasticsearch는 데이터를 중앙에 저장하여 손쉽게 확장되는 광속에 가까운 빠른 검색, 정교하게 조정된 정확도, 강력한 분석을 제공합니다.
>
> (ElasticSearch 공식 문구)

ElasticSearch는 Apache Lucene(아파치 루씬) 기반의 Java 오픈소스 분산 검색엔진이다. ElasticSearch를 통해 루씬 라이브러리를 단독으로 사용할 수 있게 되었으며, 방대한 양의 데이터를 신속하게 거의 실시간으로 저장, 검색, 분석할 수 있다.

ElasticSearch은 검색을 위해 단독으로 사용되기도 하고, ELK(ElasticSearch / logstash / kibana) 스택으로 사용되기도 한다

> ELK 스택
>
> - Logstash : 다양한 소스의 로그 또는 트랜잭션 데이터를 수집, 집계, 파싱하여 ElasticSearch로 전달
> - ElasticSearch : Logstash로부터 받은 데이터를 검색 및 집계하여 필요한 관심있는 정보를 획득
> - Kibana : ElasticSearch의 빠른 검색을 통해 데이터를 시각화 및 모니터링

![ELK 스택](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F993B7E495C98CAA706)

<br/>

## ElasticSearch와 RDBMS 용어 비교

| RDBMS              | ElasticSearch        |
| ------------------ | -------------------- |
| Database           | index                |
| Table              | Type                 |
| Row                | Document             |
| Column             | Field                |
| Index              | Analyze              |
| Primary key        | _id                  |
| schema             | Mapping              |
| Physical partition | Shared               |
| Logical Partition  | Route                |
| Relational         | Parent/Child, Nested |
| SQL                | Query DSL            |

<br/>

## ElasticSearch 아키텍처 / 용어 정리

![ElasticSearch아키텍처](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99A97A355C98D42D2E)

- 클러스터
  - ElasticSearch에서 가장 큰 시스템 단위이며, 최소 하나 이상의 노드로 이루어진 노드들의 집합이다
  - 서로 다른 클러스터는 데이터의 접근, 교환을 할 수 없는 독립적인 시스템으로 유지되며, 여러 대의 서버가 하나의 클러스터를 구성할 수 있고, 한 서버에 여러 개의 클러스터가 존재할 수도 있다
- 노드
  - ElasticSearch를 구성하는 하나의 단위 프로세스
  - 역할에 따라 다음과 같이 구분된다
    - Master-eligible node : 클러스터를 제어하는 마스터로 선택할 수 있는 노드.
      - 인덱스 생성, 삭제
      - 클러스터 노드들의 추적, 관리
      - 데이터 입력 시 어느 샤드에 할당할 건지 선택
    - Data node : 데이터와 관련된 CRUD 작업을 하는 노드. CPU, 메모리 등 자원을 많이 소비하므로 모니터링이 필요하며 master 노드와 분리되는 것이 좋다
    - ingest node : 데이터를 변환하는 등 사전 처리 파이프라인을 실행
    - coordination only node : 로드밸런서와 같이 data node와 master-eligible node의 일을 대신한다
- 인덱스 : RDBMS의 데이터베이스와 대응하는 개념. 
- 샤드(Shard) 
  - 데이터를 분산해서 저장하는 방법. 스케일 아웃을 위해 인덱스를 여러 샤드로 쪼갠다. 
  - 기본적으로는 한개가 존재하며, 검색 성능 향상을 위해 클러스터의 샤드 갯수를 조정하는 튜닝읗 하기도 한다.
  - 샤드의 종류
    - 프라이머리 샤드 : 데이터의 원본. ElasticSearch에서 데이터 업데이트 요청을 날리면 반드시 프라이머리 샤드에 요청하게 되고, 해당 내용은 레플리카 샤드에 복제된다. 
    - 레플리카(Replica) 샤드
      - 프라이머리 샤드의 복제본. 노드를 손실했을 경우 데이터의 신뢰성을 위해 샤드들을 복제한다.
      - 따라서 레플리카는 서로 다른 노드에 존재할 것을 권장한다.

<img src="https://i.stack.imgur.com/6wq4N.png" alt="replica" style="zoom:150%;" />

- 세그먼트
  - ElasticSearch에서 문서의 빠른 검색을 위해 설계된 자료구조.
  - 샤드의 데이터를 가지고 있는 물리적인 파일이다. 각 샤드는 다수의 세그먼트로 구성되어있으므로 검색 요청을 분산처리하여 훨씬 효율적인 검색이 가능하다.
  - 샤드에서 검색 시, 먼저 각 세그먼트를 검색하여 결과를 조합한 후 최종 결과를 해당 샤드의 결과로 반환하게 된다. 이때 세그먼트는 내부에 색인된 데이터가 역색인 구조로 저장되어 있으므로 검색 속도가 매우 빠르다.
  - 인메모리 버퍼 
    - 매 요청마다 새로운 세그먼트를 만들면 비효율적이므로, 요청을 먼저 저장하는 버퍼이다. 인메모리 버퍼가 가득차거나 일정 시간이 지나면 flush 작업이 일어나고, 시스템 캐시에 세그먼트가 생성된다. 이 시점에서 데이터 검색이 가능해진다.
    - 시스템 캐시에 저장된 세그먼트는 일정 시간이 지나면 commit을 통해 물리적인 디스크에 저장된다.
    - 저장된 세그먼트는 시간이 지날 수록 하나로 병합하는 과정을 수행하고, 병합을 통해 세그먼트를 하나로 줄여 주면, 검색할 세그먼트의 개수가 줄어들기 때문에 검색 성능이 향상된다.

<br/>

## **ElasticSearch**의 특징

- Scale out : 샤드를 통해 규모가 수평적으로 늘어날 수 있음

- 고가용성 : Replica를 통해 데이터의 안정성을 보장

- Schema Free : Json 문서를 통해 데이터 검색을 수행하므로 스키마 개념이 없음

- Restful : 데이터 CRUD 작업은 HTTP Restful API를 통해 수행하며 각각 다음과 같이 대응한다

  | CRUD   | Elasticsearch Restful |
  | ------ | --------------------- |
  | SELECT | GET                   |
  | INSERT | PUT                   |
  | UPDATE | POST                  |
  | DELETE | DELETE                |

<br/>

## ElasticSearch의 역색인

아파치 루씬은 기본적으로 역색인이라는 구조로 데이터를 저장한다. ElasticSearch도 루씬 기반이므로 역색인을 사용한다.

역색인(inverted index)이란, 책의 맨 뒤에 키워드가 어떤 페이지에 있는지 알려주는 페이지처럼, 값을 통해 위치를 알아내는 방식을 통해 RDBMS보다 전문 검색(full text search)에 빠른 성능을 보인다.

예를 들어 "Lorem Ipsum is simply dummy text of the printing and typesetting industry" 란 문장이 있으면 이 문장을 모두 파싱해서 각 단어들을 저장하고, 대문자는 소문자 처리하고, 유사어도 체크하여 텍스트를 저장한다. 그리고 키워드와 함께 검색 요청이 오면, 키워드를 통해 문서를 찾아내어 검색 성능이 매우 빠르다.

<br/>

## ElasticSearch 단점

- 실시간 처리가 불가능하다
  - ElasticSearch은 인메모리 버퍼를 사용하므로, 쓰기가 발생해도 세그먼트 생성 전까지는 해당 데이터를 검색할 수 없다
- 트랜잭션을 지원하지 않는다
  - 분산 시스템 구성의 특징때문에 시스템적으로 비용 소모가 큰 트랜잭션 및 롤백을 지원하지 않는다
- 진정한 의미의 업데이트를 지원하지 않는다.
  - 세그먼트에서 데이터가 삭제되면 soft delete를 한다
  - 세그먼트에서 데이터가 수정되면 soft delete를 하고 수정된 데이터를 새로운 세그먼트로 생성한다
  - RDBMS의 index와 유사한 동작

<br/>

## NoSql과의 차이

|                 | ElasticSearch | NoSQL         |
| --------------- | ------------- | ------------- |
| 실시간 처리     | 불가능        | 가능          |
| 역색인 자료구조 | 있음          | 없음          |
| 구조            | 검색 엔진     | 데이터 저장소 |

<br/>

## 기타 참고할 문서

- [ElasticSearch 한국어 가이드](https://esbook.kimjmin.net/01-overview/1.1-elastic-stack/1.1.1-elasticsearch)
- [역색인이란?](https://steady-coding.tistory.com/581)

<br/>

## 출처

https://steady-coding.tistory.com/573

https://victorydntmd.tistory.com/308