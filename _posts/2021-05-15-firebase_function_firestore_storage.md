---
title: Firebase/Function firestore, storage 쓰기, 읽기
author: 신성일
date: 2021-05-15 16:00:00 +0900
categories: [firebase, functions]
tags: [firebase, functions, crawl, firestore]
---

firebase cloud function에서 firestore와 storage의 이벤트트리거를 등록하는 예제는 firebase 문서에 자세히 설명되어있다.

여기서는 cloud functions에서 firestore와 storage에 접근하여 읽고 쓰는 법을 간단한 코드로 보여줄 것이다.

- firestore
  1. firebase cloud function 가이드에 나온대로 admin을 초기화시켜준다.
  2. admin.firestore() 로 firestore 관련 함수를 호출하면 끝.

```js
const admin = require("firebase-admin");
admin.initializeApp();

...
// 한 document 가져오기
const doc = await admin
         .firestore()
         .collection("컬렉션이름")
         .doc("document이름")
         .get();
// document set
await admin
         .firestore()
         .collection("컬렉션이름")
         .doc("document이름")
         .set({
            url: req.body.url,
            name: req.body.name,
            createdAt: Date.now(),
         });
...
```

- storage

  1. @google-cloud/storage 에서 Storage 모듈 가져오기

  2. Storage 객체를 생성

  3. storage.bucket으로 bucket url로 bucket reference 생성

  4. 생성한 bucket reference로 file reference를 만들고, save 메소드를 통해 저장

     ```js
     const { Storage } = require("@google-cloud/storage");

     const storage = new Storage();

     ...
     const bucket = storage.bucket("버켓 url");

     //file ref 생성
     const file = bucket.file(`${req.body.uid}/${req.body.uuid}.png`);

     //저장
     await file.save(screenshotBinary, {
     	metadata: { contentType: "image/png" },
     });
     ...
     ```
