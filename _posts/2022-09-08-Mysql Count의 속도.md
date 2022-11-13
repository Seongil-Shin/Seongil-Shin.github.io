---
title: Mysql Count 속도
author: 신성일
date: 2022-09-08 18:19:26 +0900
categories: [study, database]
tags: [database, db, mysql, count]
---



## **Mysql의 Select Count(\*) 얼마나 빠를까?**

사용하는 데이터베이스 엔진에 따라 다르다.

- InnoDB : O(n)
- MyISAM : O(1)

이유는 MyISAM의 경우 row가 몇 개 있는지 저장하고 있기에 `Count(*)` 쿼리가 들어오면 그 저장한 값을 바로 리턴하여 O(1)이 된다.

<br/>

## **Count(\*) vs Count(column) vs Count(\*) Where** 

`Count(*)`이 가장 빠르다.

이유는 `Count(*)` 의 경우 내부적으로 값을 확인하지 않고, row의 개수만 확인한다.

하지만 column을 주면 내부적으로 값을 확인하여 null이 아닌 row만 리턴하기에 시간이 좀 더 걸리게 된다.

where 조건도 마찬가지다. 값을 확인한다음 조건에 맞는 것만 세기에 `Count(*)`보다 느리다.

이는 `MyISAM` 엔진에서도 마찬가지다. MyISAM 엔진에서 빠른 것은 `SELECT COUNT(*)`이고 `Count(column)`이나 `Count(*) Where` 은 InnoDB와 마찬가지로 값을 읽고 리턴하기에 시간이 오래 걸린다.

<br/>

## **정리**

|                | MyISAM | InnoDB |
| -------------- | ------ | ------ |
| Count(*)       | 빠름   | 보통   |
| Count(col)     | 느림   | 느림   |
| Count(*) Where | 느림   | 느림   |

절대적으로 빠름, 느림을 의미하는게 아닌 상대적인 속도를 의미.

<br/>



## **출처**

https://stackoverflow.com/questions/5257973/mysql-complexity-of-select-count-from-mytable

https://m.blog.naver.com/pjt3591oo/221030483713

https://stackoverflow.com/questions/22352073/improve-innodb-count-performance

https://imnkj.tistory.com/49