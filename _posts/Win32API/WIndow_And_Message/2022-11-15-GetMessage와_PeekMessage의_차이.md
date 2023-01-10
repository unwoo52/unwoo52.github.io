---
title: GetMessage와 PeekMessage의 차이
author: unwoo52
date: 2022-11-15 00:00:00 +09:00
categories: [Win32API]
tags: [Win32API]
---

# GetMessage와 PeekMessage

우리가 키나 마우스 등의 입력을 가하게 되면 입력이 메세지 큐에 쌓이게 되고, 이를 가져오는 함수가 GetMessage와 PeekMessage이다.
두 함수는 동일한 매개변수 4개를 갖는다.

```
BOOL GetMessage(
  [out]          LPMSG lpMsg,
  [in, optional] HWND  hWnd,
  [in]           UINT  wMsgFilterMin,
  [in]           UINT  wMsgFilterMax
);
```

PeekMessage는 여기에 하나 더해서 5개의 매개변수를 갖는다.

```
BOOL PeekMessageA(
  [out]          LPMSG lpMsg,
  [in, optional] HWND  hWnd,
  [in]           UINT  wMsgFilterMin,
  [in]           UINT  wMsgFilterMax,
  [in]           UINT  wRemoveMsg
);
```

*lpMsg : MSG구조체에 대한 포인터*

*hWnd : 메세지를 검색할 윈도우에 대한 핸들. 만약 NULL일경우 현재 스레드에 속한 모든 윈도우에 대한 메세지와
hwnd값이 NULL인 현재 스레드의 메시지 큐에 있는 모든 메시지를 검색한다.*

*wMsgFilterMin : 검색할 가장 낮은 메시지 값의 정수 값.*

*wMsgFilterMax : 검색할 가장 높은 메시지 값의 정수 값.이 두 인수를 사용해 일정 범위에 속하는 메세지만 조사하게 할 수 있다. 둘 다 0이면 모든 메세지를 조사한다.*

*wRemoveMsg : PM_NOREMOVE이면 메세지를 읽은 후 큐에서 메세지를 제거하지 않는다. PM_REMOVE이면 메세지를 읽은 후 큐에서 메세지를 제거한다.*


**결론부터 말하면 게임을 만들기에 좋은 쪽은 PeekMessage이다.**

## GetMessage

 GetMessage는 메세지가 없으면 메세지가 발생할 때 까지 활동하지 않고 대기한다. 세 메시지가 올 때 까지 블록되어 CPU를 사용하지 않는다.
 
 하지만 게임은 입력이 없어도 끊임없이 돌아가야 한다. 입력이 없으면 화면이 정지되는 게임이 되면 곤란할 것이다.
 
## PeekMessage

PeekMessage는 메세지큐가 비어있다면 기다리는 활동 없이 0을 리턴한다. 때문에 메시지큐가 비어있어도 다른 작업이 작동할 수 있다.
