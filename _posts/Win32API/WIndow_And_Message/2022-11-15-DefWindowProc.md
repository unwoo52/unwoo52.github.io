---
title: DefWindowProc
author: unwoo52
date: 2022-11-15 00:00:00 +09:00
categories: [Win32API]
tags: [Win32API]
---

## DefWindowProc

윈도우프로시저에서 처리한 이후, 처리되지 않은 나머지 메세지들을 처리해준다.

```
LRESULT CALLBACK WndProc(HWND hWnd,UINT iMessage,WPARAM wParam,LPARAM lParam)
{

	switch(iMessage) {
	case WM_CREATE:
		return 0;
	case WM_PAINT:
		hdc=BeginPaint(hWnd, &ps);
		EndPaint(hWnd, &ps);
		return 0;
	case WM_COMMAND:
		return 0;
	case WM_DESTROY:
		PostQuitMessage(0);
		return 0;
	}
	return(DefWindowProc(hWnd,iMessage,wParam,lParam));
}
```

모든 처리를 마친 이후 마지막으로 DefWindowProc으로 return하는 것을 볼 수 있다.

단, WM_DESTROY 메시지에 대해 PostQuitMessage는 호출해 주지 않는다.

