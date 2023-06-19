---
title: IOS 16.5에서 video가 크기 변경이 적용되지 않는 이슈
author: 신성일
date: 2023-06-18 18:00:00 +0900
categories: [study, js]
tags: [issues]
---



## Spec

- [Swiper](https://swiperjs.com/) (ver 4.5.1) 안에 `<video>` 태그가 `slide`로 포함되어있음.

- 현재 가장 가운데 있는 slide 안의 video는 크기를 키우고, slide가 옆으로 이동하면 다시 원래 사이즈로 돌아가도록 함

  <video src="/assets/video/2023-06-18-ios-issue-spec.mov"></video>

## Issue

- IOS 16.5 safari에서는 다음과 같이 가운데 slide 안의 video 크기가 정상적으로 확대되지 않음

<video src="/assets/video/2023-06-18-ios-issue.mp4"></video>

## 원인

- 가운데 video 확대를 위한 코드는 다음과 같이 `slide-active` 상태일때 스타일을 적용하여 크기를 변경해주는 로직이다.

  ```css
  .swiper .swiper-slide video {
      width: 173px;
      height: 375px;
  }
  
  .swiper .swiper-slide-active video {
      width: 238px !important;
      height: 418px !important;
  }
  ```

  - 가운데 slide에 정상적으로 `swiper-slide-active` 클래스가 적용되면 video 태그의 크기가 정상적으로 커져야했고, 개발자도구로 확인했을떄 문제가 발생했던 기기에서도 제대로 클래스가 적용된 것을 확인할 수 있었다. 하지만 이유는 알수없지만  ios 16.5 safari에서는 video 태그의 크기변화를 잡아내지 못하여 화면에 적용되지 않은 것이다.

## 해결

다음과 같이 swiper 이벤트 중 `slideChangeTransitionStart`가 발동될 떄 명시적으로 style을 변경하고, 약간의 타이머를 주어 1px 크게 변경하여 브라우저가 크기 변화를 인식할 수 있도록 하였다. 

```js
slideChangeTransitionStart : function() {
	slides?.forEach(function(item) { 
		if(item.slide.classList.contains("swiper-slide-active")) {
			if(window.isIOS) {
        if(window.innerWidth >= 1024) {
       	 activeVideo.style.setProperty("width", "236px", "important");
     	   activeVideo.style.setProperty("height", "516px", "important");
        } else {
     	   activeVideo.style.setProperty("width", "238px", "important");
     	   activeVideo.style.setProperty("height", "418px", "important");
        }	
        
				setTimeout(() => {
					if(window.innerWidth >= 1024) {
						activeVideo.style.setProperty("width", "237px", "important");
						activeVideo.style.setProperty("height", "517px", "important");
					} else {
						activeVideo.style.setProperty("width", "239px", "important");
						activeVideo.style.setProperty("height", "419px", "important");
					}
				}, 30)
			}
		} else {
				if(window.isIOS) {
					if(window.innerWidth >= 1024) {
						item.video.style.setProperty("width", "187px", "important");
						item.video.style.setProperty("height", "405px", "important");
					} else {
						item.video.style.setProperty("width", "187px", "important");
						item.video.style.setProperty("height", "328px", "important");
					}
          
          setTimeout(() => {
            if(window.innerWidth >= 1024) {
              item.video.style.setProperty("width", "188px", "important");
              item.video.style.setProperty("height", "406px", "important");
            } else {
              item.video.style.setProperty("width", "188px", "important");
              item.video.style.setProperty("height", "329px", "important");
            }
          }, 30)
				}
			}
		})
},
```

이후 문제없이 동작함을 확인하였다.
