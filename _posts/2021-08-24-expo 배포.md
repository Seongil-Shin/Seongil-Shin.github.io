---
title: expo 배포
author: 신성일
date: 2021-08-24 16:00:00 +0900
categories: [study, react-native]
tags: []
---

## EXPO 앱 google playstore 배포

1. app.json 수정
2. expo build:android -t app-bundle
   - google play app signing 이 선행되어야함.
     - https://docs.expo.dev/distribution/app-signing/
     - keystore를 구글 개발자계정에서 받을 수 있는듯?
     - https://support.google.com/googleplay/android-developer/answer/9842756
   - keystore는 generate하되, expo fetch:android:keystore를 통해 가지고 있자.
     - 업데이트를 같은 keystore로 해야함.
3. 끝나면 .apk 또는 .aab 파일이 나옴
   - .apk 면 에뮬레이터에 넣어서 테스트함.
4. playstore에 제출
   - 처음 제출일 경우 https://github.com/expo/fyi/blob/master/first-android-submission.md
   - 이후엔 eas 사용하면 되는듯
     - https://docs.expo.dev/distribution/uploading-apps/
5. 업데이트
   - expo publish 하면 자동으로 업데이트됨.
   - 하지만 다음과 같은 이유로 다시 제출하고 싶을때가 있음
     - native metadata를 바꾸고 싶을때 (ex : app's name, icon)
     - 최신 SDK 버전으로 업그레이드하고 싶을때.
   - 그럴때는 versionCode 또는 buildNumber를 바꿔서 업데이트해줘야함
