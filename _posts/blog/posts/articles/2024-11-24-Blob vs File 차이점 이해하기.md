---
title: Blob vs File 차이점 이해하기
author: 신성일
date: 2024-11-24 19:00:00 +0900
categories: study, article
tags:
  - "#study"
---
https://jsdev.space/file-blob-js/

## Blob

Blob는 raw binary data (file, images, videos, non-text data 등)를 다루기 위해 주로 사용되는 자바스크립트 불변 객체이다

특징
1. 불변성 : 한번 만들면 수정이 불가능하다. 다른 데이터와 합쳐서 새로운 Blob를 만드는 것은 가능하다
2. Data Representation : 어떤 타입의 데이터도 다룰수 있다. 보통 웹 어플리케이션에서 이진 데이터를 다루기 위해 사용된다
3. size, type 이라는 두가지 속성이 존재한다
	1. size : blob의 byte 크기
	2. type : MIME type
4. 다양한 Web API에 사용된다
	1. File API : 유저가 업로드한 파일을 다룸
	2. canvas API : canvas 요소에서 이미지를 생성할 때 사용함
	3. Fetch API : 이진데이터를 송수신하는데 사용됨
5. Blob 생성 : Blob 생성자로 생성가능
	1. `new Blob(["hello world"], { type: "type/plain" })`


## File

유저의 파일 시스템에서 가져온 파일의 표현방식. Blob의 특수한 형태로 file에 대한 정보를 담고 있음 (name, size, type, mod date). 주로 웹에서 파일 업로드 등을 다룰때 사용한다

특징
1. Blob으로부터 상속됨. 따라서 Blob의 정보 및 파일에 관한 정보가 추가로 들어있음.
2. name, size, type, lastModified, webkitRelativePath 속성 포함
	1. webkitRelativePath : 디렉터리까지의 상대경로. 보통 디렉터리 내 여러 파일을 선택했을때 사용됨
3. 유저가 html input element로 파일을 선택했을때 주로 생성됨. 
4. FileReader API 로 데이터를 다룰 수 있다
5. fetch 등을 통해 서버로 보낼 수도 있음.

