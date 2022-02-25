---
title : DataStructure 개요
author : 신성일
date : 2022-01-23 14:00:00 +0900
categories : [cs, datastructure]
tags: [DataStructure]
---





## Array vs Linked List

**Array**

- 논리적 저장 순서와 물리적 저장순서가 일치함.

- 장점

  - Random Access : 인덱스로 원하는 원소에 바로 접근이 가능하여, O(1)에 해당원소로 접근가능

  - 언어에따라 다르지만, 필요한 만큼만 메모리를 차지함

- 단점

  - 중간에 한 원소를 삭제하거나, 새로운 원소를 넣을때, 다른 원소를 shift 해줘야하여 O(n)의 시간이 걸림.

**Linked List**

- 각각의 원소가 자기 다음 (그리고 이전) 원소가 무엇인지만 알고있음
- 장점
  - 삭제/삽입시 영향을 받는 다음/이전 원소의 값만 변경해주면 되기에 O(1)의 시간이 걸림
- 단점
  - 논리적 저장순서와 물리적 저장순서가 일치하지 않아서, 원소를 찾을때는 처음 원소부터 차례대로 찾아야하기에 O(n)의 시간이 걸린다.
  - 이때 삭제/삽입 시에도, 원소를 찾는 과정이 필요하기에 결국 삭제/삽입 시에도 O(n)이 걸리게 된다.





그럼 왜 Linked List를 쓰는가? -> 다른 로직과 결합한다면 삭제/삽입/찾기의 시간복잡도를 줄일 수 있고, Tree의 근간이 되는 자료구조이기에







## Tree

비선형 자료구조로, 계층적 관계를 표현하는 자료구조이다. 

트리는 `표현`에 집중하며, 무엇인가를 저장하고 꺼낸다기보다는 어떤 데이터의 구조를 표현하는 것에 집중한다.

**트리 용어**

- Node : 트리를 구성하는 원소들
- Edge : 트리를 구성하기 위해 노드와 노드를 연결하는 선
- Rood Node : 최상위 노드
- Terminal Node(leaf node) : 하위에 다른 노드가 연결되지 않은 노드
- Internal Node (내부노드) : 단말 노드를 제외한 모든 노드(루트 포함)
- Level : 0(루트노드)부터 시작하여, 트리의 층별로 레벨을 하나씩 부여한 것
- height : 해당 트리의 최고레벨 



**Binary Tree**

한 부모가 두개의 자식만 가질 수 있는 트리를 말한다. 공집합도 이진트리에 포함된다. 

- 포화이진트리 : 모든 레벨이 꽉 찬 이진트리
- 완전 이진트리 : 위에서 아래로, 왼쪽에서 오른쪽으로 순서대로 채워진 이진트리
- 정 이진트리(Full Binary Tree) : 모든 노드가 0개 혹은 2개의 자식만을 가진 이진트리

배열로 구성된 Binary Tree는 root가 0이 아닌 1에서 시작할때, 부모노드의 인덱스가 i라고 한다면, 왼쪽 자식은 2i, 오른쪽 자식은 2i + 1이다.





**Binary Search Tree**

이진탐색트리는 이진트리의 일종으로, 데이터의 탐색을 용이하게 하기 위한 데이터 저장방법으로 고안되었다.

이진탐색트리에 데이터를 저장하는 규칙

- 이진탐색트리의 노드에 저장된 키는 유일하다
- 부모의 키가 왼쪽자식 노드의 키보다 크다
- 부모의 키가 오른쪽자식 노드의 키보다 작다.
- 왼쪽과 오른쪽 서브트리도 이진탐색트리다.

이진탐색트리에서 탐색연산은, 높이만큼 탐색하기에 O(h)이며, 완전이진트리일 경우 O(log n)의 시간복잡도를 가진다. 하지만 worst case로 데이터가 편향되어 트리에 들어왔을 경우 한쪽으로 치우치게 되어 O(n)의 복잡도를 가지게 된다.

이를 해결하기 위해 `Rebalancing` 기법을 사용하여, 트리의 균형을 다시 잡는 작업을 하기도 한다. 이 기법을 구현한 트리로는 `Red-Black Tree`가 있다.





##  Binary Heap

완전이진트리에 기반한 트리의 일종이다. `최대힙`, `최소힙` 이 있다.

- 최대힙 : 각 노드의 값이 자식의 값보다 크거나 같은 완전이진트리. 최댓값을 찾는데 사용
- 최소힙 : 각 노드의 값이 자식의 값보다 작거나 같은 완전이진트리. 최솟값을 찾는데 사용

삽입/삭제 연산은 최대 층수까지만 하니, O(log n)이다.





**구현**

1. 루트노드는 1부터 시작.
2. 삽입 연산
   - 트리의 가장 마지막에 삽입
   - 부모와 값을 비교해가며 올라감
   - 최소힙일 경우, 부모보다 클경우 멈춤
3. 삭제 연산
   - 루트 노드가 사용자의 요청으로 사라짐
   - 가장 끝의 노드를 루트노드에 삽입
   - 두 자식보다 값이 작을 때까지 아래로 내려감(둘 중 하나만 본인보다 작을 경우 그쪽과 교환. 둘 다 작을 경우 아무데로)





## Red Black Tree

이진 탐색트리에서, Rebalancing 기법을 적용하여 탐색/삭제/삽입에 O(log n)의 시간이 걸리도록 한 트리이다.

즉, 최대 높이를 `log n`로 만드는 것으로 완전이진트리에 기반을 둔다.

**Red Black Tree 정의**

RBT는 아래 성질을 만족하는 이진탐색트리이다.

- 각 노드는 Red or Black이라는 색깔을 갖는다.
- Root Node의 색상은 Black이다.
- 각 leaf node(NIL)는 black이다.
- 어떤 노드의 색깔이 red면, 두 자식은 모두 black이다.(Red가 두번 연속 나올 수 없음)
- 같은 층에 있는 각 노드에 대하여 노드로부터 descendant leaves 까지의 단순 경로는 모두 같은 수의 black nodes들을 포함하고 있다. 이를 해당 노드의 black-height라고 한다.

**Red Black tree의 특징**

- BST의 특징을 모두 갖는다.
- Root node부터 leaf node까지의 모든 경로 중 최소 경로와 최대 경로의 크기 비율은 2보다 크지 않다. 이러한 상태를 balanced라고 한다.
- 노드의 child가 없을 경우 child를 가리키는 포인터는 NIL 값을 저장한다. 이 NIL들을 leaf node로 간주한다.

![No Image](/assets/img/2022-01-22-datastructure/2.png)



**삽입**

1. BST(이진탐색트리)의 특성을 유지하면서 삽입한다.
2. 삽입된 노드에 Red를 부여한다. 
3. RBT의 특성을 위배했을 시 노드의 색깔을 조정하고, Black-height가 위배되었다면 rotation을 통해 height를 조정한다.
   - 다른 설명
     - Recoloring : 삽입된 노드의 부모의 형제 색깔이 Red일 경우
       - 부모와 부모의 형제 노드는 Black으로, 부모의 부모는 Red로(단, 루트노드일경우 Black으로 놔둠
       - 다시 double red가 발생했을 경우 다시 작업
     - Restructuring : 삽입된 노드의 부모의 형제 색깔이 Black 또는 NULL인 경우
       - 삽입된 노드, 부모, 부모의 부모를 오름차순으로 정렬
       - 중앙 값을 부모노드로 만들고, 나머지 노드를 자식으로 변환함.
       - 부모노드가 된 노드를 Black, 나머지 노드를 Red로 coloring 함
4. 이러한 과정을 통해 RBT의 동일한 height에 존재하는 internal node들의 black-height가 같아지게 되고, 최소 경로와 최대경로의 크기 비율이 2 미만으로 유지된다.

**삭제**

1. BST의 특성을 유지하면서 삽입한다.
2. 만약 지워진 노드가 black이면, 
   1. black-height가 1 감소한 경로에 black node가 1개 추가되도록 rotation하고 노드의 색깔을 조정한다.
3. 지워진 노드가 Red라면,
   1. 특성이 위배되지 않으므로 RBT가 그대로 유지된다.





## Hash Table

해시는 내부적으로 배열을 사용하여 데이터를 저장하기 때문에 빠른 검색속도를 가진다. 특정한 값을 Search할때 데이터 고유의 인덱스(해시값)으로 접근하므로, 탐색연산의 시간복잡도는 평균 O(1)이다.(충돌 때문에 항상 O(1)은 아님)

이 인덱스로 사용될 해시값은 특별한 알고리즘을 통해 저장할 데이터와 연관된 고유한 숫자를 만들어 낸 뒤 이를 인덱스로 사용한다. 이 인덱스는 그 데이터만의 고유한 위치이기에 삽입/삭제 시 다른 데이터로 채울 필요가 없으므로 연산에서 추가적인 비용이 없도록 만들어진 구조이다.



### **Hash Function**

해시값을 만들어내기 위한 함수. 이 함수에 의해 반환된 데이터의 고유숫자값을 hashcode라고 하며, 저장되는 값들의 key값을 hash function을 통해서 작은 범위의 값들로 바꿔준다.

이때 서로 다른 데이터에 동일한 key 값을 도출해낼 수 있고, 이러한 상황을 `Collision`이라고 한다.

**Collision** : 서로 다른 두개의 키가 같은 인덱스로 hasing 되면 같은 곳에 저장할 수 없게 된다. collision이 많아질수록 탐색에 필요한 시간복잡도는 O(n)에 가까워진다. 





### **Resolve Conflict**

**Open Address(개방 주소법)**

다른 해시 버킷에 충돌이 발생한 자료를 삽입하는 방식.

`다른 해시버킷`을 찾는방법 : 최악의 경우 원래 자리로 돌아옴

	1. Linear Probing : 순차적으로 탐색하여 첫번째로 발견한 빈 곳에 넣음
	1. Quadratic probing : 2차함수를 이용해 찾음
	1. Double hashing probing : 2차 해시 함수를 사용해 새로운 주소를 할당함. 많은 연산량이 필요하다.

**Separate Chaining 방식 (분리연결법)**

1. 연결리스트를 사용하는 방식
   - 각각의 버킷들을 연결리스트로 만들어, Collision이 발생하면 해당 버킷의 리스트의 끝에 삽입하는 방식
   - 삽입/삭제가 간단함. Open Address 방식에 비해 테이블의 확장을 늦출 수 있다.
   - 작은 데이터를 저장할때 연결리스트 자체의 오버헤드가 부담이 된다는 단점이 있음.
2. Tree를 사용하는 방식(RBT)
   - 위 방식과 동일한데 트리를 사용하는 것

연결리스트 vs Tree

- 해시 버킷에 할당된 key-value 쌍의 개수로 어느 것을 사용할지 결정함
- 데이터의 개수가 적다면 연결리스트를 사용한다. 트리는 메모리 사용량이 많기 때문이다.
- 이때의 기준은 6개, 8개이다. (6개, 7개일경우 데이터가 하나삽입/하나 삭제 될때마다 자료구조를 바꿔야함. 따라서 2칸의 여유를 둠)
- 예시
  - 해시버킷에 6개의 key-value 쌍이 있는데 하나의 값이 추가됨. -> 연결리스트 유지
  - 다시 하나의 값이 삭제 됨 -> 연결리스트 유지
  - 값이 다시 2개가 추가됨 -> 트리로 변경
  - 하나가 삭제 됨 -> 트리 유지
  - 하나가 삭제 됨 -> 연결리스트로 변경

**보조해시함수**

key의 해시값을 변형하여 해시 충돌 가능성을 줄이는 것. Separate chaining방식을 사용할때 함께 사용되며, 보조해시함수로 worst case에 가까워 지는 경우를 줄일 수 있다.





**Open address vs Separate Chaining**

둘다 worst case에서는 O(M)임. 하지만 Open Address 방식은 연속된 공간에 데이터를 저장하기 때문에 Separate Chaining에 비해 캐시 효율이 높다. 따라서 데이터가 충분히 적으면 Open Adderess가 좋다. 

하지만 Open Address는 버킷을 계속 사용하기에 테이블의 확장이 자주 일어난다.





**해시 버킷 동적확장(resize)**

해시버킷의 개수가 적으면 해시 충돌로 성능상 손실이 발생함. 따라서 key-value 쌍 데이터 개수가 일정 개수 이상이 되면 해시 버킷의 개수를 두배로 늘린다. 이렇게 하면 해시 충돌로 인한 성능저하를 어느정도 해결할 수 있다.

이때 두배로 확장하는 임계점은 기존 해시버킷 개수의 75%이다.





## Graph

**Undirected Graph 와 Directed Graph (Digraph)**

- Directed Graph : 방향성이 포함된 그래프
- Undirected Graph : 방향성이 없는 그래프

**Degree**

각 정점에 연결된 Edge의 개수. 방향그래프일경우 Degree가 둘로 나뉜다.

**가중치 그래프**

간선이 가중치를 두어서 구성한 그래프

**부분그래프**

본래 그래프의 일부 정점 및 간선으로 이루어진 그래프





### **그래프 구현법**

**인접행렬**

- 해당하는 위치의 value를 통해 정점간의 연결정보를 O(1)로 파악할 수 있음.
- Edge의 개수와 무관하게 V^2의 메모리를 차지함
- Dense Graph를 표현할때 적합함

**인접 리스트** : 연결리스트

- vertex의 리스트를 확인해야하기 때문에, 정점간 연결 확인에 오래 결린다.
- 메모리는 O(E + V)를 차지함
- Sparse Graph를 표현할때 적합함.





### **그래프 탐색**

**DFS**

- 연결되어있는 정점으로 계속 가다가 더이상 연결된 정점이 없으면 되돌아와서 다시 탐색함
- O(V +E)

**BFS**

- 한 정점에 연결되어있는 모든 정점으로 나아감
- O(V + E). BFS로 구한 경로는 루트로부터 해당 노드로의 최단 경로임





### **최소신장트리(MST)**

**크루스칼**

가중치가 가장 낮은 edge를 하나씩 추가하되, 사이클을 형성하지 않는 edge를 추가하는 것. V - 1개의 Edge가 추가되면 정지함.

시간 복잡도

1. Edge의 weight를 기준으로 sorting - O(ElogE)
2. cycle 생성여부를 검사하고 id를 통일 -> O(E + V log V)

**Prim**

현재까지의 그래프와 아직 추가되지 않은 노드와 연결되어 있는 간선 중에서 가장 작은 가중치를 가지는 간선만 추가함.

시간복잡도 : O(E logv)