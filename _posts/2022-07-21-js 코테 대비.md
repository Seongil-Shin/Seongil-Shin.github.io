---
title : JS 코테 대비 메소드 정리
author : 신성일
date : 2022-07-21 22:00:00 +0900
categories : [알고리즘]
tags: []
---



## 문법

- for ... of : 반복가능한 객체 (Array, String, Set)
- for ... in : 객체. 키를 뽑아냄.

<br/>

## Set

- Set.prototype.**add**(value)
- Set.prototype.clear()
- Set.prototype.delete(value)
- Set.prototype.forEach((value, key, set) => {})
  - 값, 키, set을 받음
- Set.ptototype.has(value)

<br/>

## Map

- Map.prototype.**set**(key, value)

- Map.prototype.get(key)

- Map.prototype.delete(key)

- Map.prototype.has(key)

- Map.prototype.forEach((value, key, map) => {})

<br/>

## Array

- **Array.prototype.pop**()
  - 마지막 요소를 제거하고 그 요소를 반환함.
- **Array.prototype.reverse()**
  - 배열을 반전시킴
- **Array.prototype.shift()**
  - 배열의 첫번째 요소를 제거하고, 제거된 요소를 반환함. 
- **Array.prototype.unshift()**
  - 배열의 맨 앞쪽에 추가하고, 새로운 길이를 반환함.
- Array.prototype.slice(begin?, end?)
  - begin부터 end까지 잘라서 반환. end가 length보다 길면 length까지만 반환
- **Array.prototype.sort(compareFunction)**
  - compareFunction 반환값
    - < 0 : a를 b보다 낮은 색인으로 정렬
    - = 0 : 변경하지 않음
    - \> 0 : b를 a보다 낮은 색인으로 정렬
- **Array.prototype.splice(start, deleteCount?, item1?, ....)**
  - start부터 deleteCount만큼 삭제하고, 해당 위치에 item...을 추가
  - 원본을 변경시킴.

<br/>

## String

- String.prototype.includes(word)
  - string 안에 word가 존재하는지 검사
- String.prototype.slice(start, end?)
  - start 부터 end까지 잘라서 반
- reverse : str.split("").reverse().join("");

<br/>

## Number

- Number.MAX_VALUE

- **Number.parseInt**(string)

<br/>

## Math

- Math.ceil()
- Math.floor()
- Math.trunc() : 소수부분을 제거하고 정수부분을 반환. (음수, 양수 상관없이)

<br/>

## Object

- Object.entries(object)

  - [키, value] 형태로 배열을 반환

    ```js
    const object1 = {
      a: 'somestring',
      b: 42
    };
    
    for (const [key, value] of Object.entries(object1)) {
      console.log(`${key}: ${value}`);
    }
    ```

<br/>

<br/>

## lower_bound, upper_bound 원리

**lower_bound**

- mid의 값이 k 미만일때 : [mid + 1, end] 를 탐색
- 그 외 : mid가 lower bound가 될 수 있으므로, 결과 변수에 기존값과 mid를 비교해 더 작은 값을 저장하고, lower bound가 될 수 있는 더 작은 값이 있는지 [start, mid - 1]를 탐색한다.

**upper_bound**

- mid의 값이 k 이하일때 : [mid + 1, end] 탐색
- 그 외 : mid가 upper_bound 가 될 수 있으므로, 결과 변수에 기존 값과 mid를 비교해 더 작은 값을 저장하고, 더 작은 구간 [start, mid - 1]을 탐색한다.
