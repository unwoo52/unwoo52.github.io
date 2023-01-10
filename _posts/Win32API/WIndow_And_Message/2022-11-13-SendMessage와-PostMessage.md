---
title: SendMessage와 PostMessage
author: unwoo52
date: 2022-11-13 00:00:00 +09:00
categories: [Win32API]
tags: [Win32API, Message]
---

## SendMessage

 윈도우에 지정된 메세지를 보낸다. 대상 윈도우에 대한 메세지프로시저를 호출하고, 메세지가 처리되기 전까지 return을 하지 않는다.
 
 때문에 메세지를 보내고 메세지가 처리되기 전까지 블록상태로 있는다.
 
 장점은 메세지의 확실한 처리가 보장된다는 점
 
 단점은 메세지를 보낸 쪽이 블록상태로 머문다는 것이고, 기아상태에 빠져 영영 빠져나오니 않을 수도 있다.
 
 때문에 메인스레드에서 다른 스레드로 메세지를 보낼 때 SendMessage를 사용하는 것은 위험하다.

## PostMessage

 윈도우에 지정된 메세지를 보낸다. 대상 윈도우에 대한 메세지프로시저를 호출하고, 메세지를 처리하는 것을 기다리지 않고 바로 반환한다.
 
 장점은 메세지를 보내는 쪽에서 블록되지 않고 바로 작업을 이어간다는 점
 
 단점은 메세지큐에 다른 메세지가 있으면 바로 처리되지 않을 수 있다.
 
 메세지가 언제 처리되는지 예측할 수 없다.

#### 참고
[https://six605.tistory.com/203](https://six605.tistory.com/203)

[https://learn.microsoft.com/ko-kr/windows/win32/wintouch/sendmessage--postmessage--and-related-functions](https://learn.microsoft.com/ko-kr/windows/win32/wintouch/sendmessage--postmessage--and-related-functions)
