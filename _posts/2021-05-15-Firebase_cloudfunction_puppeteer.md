---
title: Firebase/Function Puppeteer 사용하기
author: 신성일
date: 2021-05-15 16:00:00 +0900
categories: [study, firebase]
tags: [firebase, functions, crawl, puppeteer]
---

일반 노드에서의 환경과 같이 사용하면 된다.

- 홈페이지에 접속 후 스크린샷을 찍어 스토리지에 업로드하는 예시

```js
const puppeteer = require("puppeteer");
const { Storage } = require("@google-cloud/storage");

const storage = new Storage();

router.post("/", (req, res) => {
  (async () => {
    const browser = await puppeteer.launch({
      args: ["--no-sandbox", "--disable-setuid-sandbox"],
    });
    const page = await browser.newPage();

    await page.goto(req.body.url);

    await page.waitForTimeout(5000);
    const screenshotBinary = await page.screenshot({
      fullPage: true,
      encoding: "binary",
    });

    const bucket = storage.bucket("스토리지 버켓 토큰");

    const file = bucket.file(`${req.body.uid}/${req.body.uuid}.png`);

    await file.save(screenshotBinary, {
      metadata: { contentType: "image/png" },
    });
    await browser.close();

    res.send({ data: "uploaded" });
  })();
});
```

## 주의점

firebase cloud function은 연결된 함수가 실행되고, 종료되면 그 함수를 돌리기위한 환경또한 같이 종료되게 된다.

따라서, cloud function에서 puppeteer를 사용하려면, 작업이 전부 완료될때까지 함수가 종료되지 않도록 해줘야한다.

```js
...
return Promise.all(promises).then(async() => {
    await browser.close();
})
...
```

위처럼 puppeteer를 부르는 모든 작업을 promises에 집어넣어, 이 promise 함수들이 전부 끝나기 전까지 종료되지 않도록 하거나, await문을 적절히 사용하여 종료하지 않도록 하면 된다.
