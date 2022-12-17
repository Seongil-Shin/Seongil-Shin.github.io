---
title: junit - spy
author: 신성일
date: 2022-11-12 18:19:26 +0900
categories: [study, java]
tags: []
---

Mock 객체와는 달리 객체의 특정 메서드만 stub으로 대체할 수 있는 방법을 제공한다.

다음과 같이 만약 테스트 대상 객체의 특정한 메서드를 stub 처리하고 싶을 때 사용한다.

```java
@ExtendWith(MockitoExtension.class)
public class BoardServiceTest{  
  @Spy
  private BoardService boardService;
  
  @Test
  public void 인증_실패시_예외발생() {
    // boardService의 인증을 담당하는 메서드를 stub 처리함
    doReturn(false).when(boardService).checkAuthorization(anyLong(), anyString());

    // boardService의 update 메서드는 정상적으로 호출한 후, 예외를 뱉는지 확인
    assertThrows(ApiException.class, () -> boardService.updateArticle(any()));
  }
}
```

