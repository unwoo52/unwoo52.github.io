---
title: CreateWindow
author: unwoo52
date: 2022-11-16 00:00:00 +09:00
categories: [Win32API]
tags: [Win32API]
---


윈도우 만드는 순서

1. WNDCLASS : 윈도우 클래스 속성 설정
2. RegisterClass : 윈도우 클래스 등록
3. CreateWindow : 윈도우 인스턴스 생성
4. ShowWindow : 윈도우 인스턴스 시각화
5. while(GetMessage){} : 메세지 루프문

## WNDCLASS : 윈도우 클래스 속성 설정

먼저 처음으로 WindowClass를 생성하는데, 여기서 Class는 c++의 클래스가 아님을 인식해야 한다.
여기서의 class는 운영체제 내부적으로 사용하는 데이터 구조인 class를 의미하는 것이다. 이제 WNDCLASS라는 구조체를 정의한다.

```cs
void RenderWindow::RegisterWindowClass()
{
    //구조체
    WNDCLASSEX wc = { 0 }; // { 0 } 은 모든 값을 0으로 초기화 하게 하며, Uniform initialization라고 부른다.
    wc.style = CS_HREDRAW | CS_VREDRAW | CS_OWNDC;
    //wc.lpfnWndProc = HandleMessageSetup;
    wc.lpfnWndProc = DefWindowProc;
    wc.hInstance = hInst;
    wc.hCursor = LoadCursor(NULL, IDC_ARROW);
    wc.lpszClassName = winClass.c_str();
    wc.cbSize = sizeof(WNDCLASSEX);
    RegisterClassEx(&wc);
}
```

#### wc 인자들에 대한 설명

*ipfnWndProc : 윈도우의 메시지를 처리하는 프로시저를 가리키는 포인터.*

*hInstance : 윈도우가 포함되어 있는 인스턴스에 대한 핸들. (인스턴스는 운영체제 내에서 작동하는 우리가 만들고 있는 프로그램 자체를 지칭. 처음 생성되는 윈도우도 인스턴스로부터 생성된다.)*

*hCursor , hIcon : 커서와 아이콘 디자인*

*hbrBackground : 빈 배경을 색칠할 브러시 색에 대한 옵션*

![imagename](/assets/image/Win32API/CreateWindow/001.png)

*lpszMenuName : 메뉴의 리소스의 이름. 이 멤버가 NULL이라면 창에 기본 메뉴가 존재하지 않음*

*lpszClassName : WindowClass의 이름을 지정, 최대 길은 256이며 이보다 크면 false를 리턴*


## RegisterClass : 윈도우 클래스 등록

운영체제에 WindowClass를 등록

```
RegisterClassEx(&wc);
```


## 3. CreateWindow : 윈도우 인스턴스 생성

```cs
handle = CreateWindowEx(0,
  winClass.c_str(), winTitle.c_str(),
  WS_CAPTION | WS_MINIMIZEBOX | WS_SYSMENU,
  0, 0, // window position
  width, height, //window size
  NULL, NULL, hInst,
  nullptr); // lParam to Create window
```

*lpClassName : WNDCLASS를 지칭할 문자열*

*lpWindowName : 윈도우 타이틀바에 나타날 문자열*

*dwStyle : 생성될 윈도우의 스타일을 지정한다.*
![imagename](/assets/image/Win32API/CreateWindow/002.png)

*x, y : 윈도우가 생성될 위치*

*nWidth, nHeight : 윈도우의 수평 크기와 수직 크기를 장치 단위(픽셀)로 지정*

*nWidth, nHeight : 윈도우의 수평 크기와 수직 크기를 장치 단위(픽셀)로 지정*

*hMenu : 오버랩드 윈도우나 팝업 윈도우의 경우 메뉴의 핸들을 지정한다. 윈도우 클래스에 메뉴가 지정되지 않았을 경우 이 인수가 지정하는 핸들이 사용되며 만약 윈도우 클래스와 이 인수에 동시에 다른 메뉴가 지정되어 있으면 이 인수가 지정하는 메뉴가 우선적으로 적용된다. 차일드 윈도우의 경우 컨트롤의 ID를 지정하는데 이 ID는 차일드가 부모 윈도우(주로 대화상자)에게 통지 메시지를 보낼 때 차일드간의 구분을 위해 사용하므로 같은 부모에 속한 컨트롤끼리는 중복되는 ID를 가지지 않아야 한다.*

*hInstance : 이 윈도우를 생성하는 인스턴스 핸들을 지정한다. 이 인스턴스가 종료될 때 윈도우도 같이 파괴된다.*

*lpParam : WM_CREATE메시지의 lParam으로 전달될 CREATESTRUCT 구조체를의 포인터이다. MDI 클라이언트 윈도우를 만들 때는 CLIENTCREATESTRUCT 구조체의 포인터를 전달해야 한다.*


## 4. ShowWindow : 윈도우 인스턴스 시각화

```
ShowWindow(handle, SW_SHOW);
```

## 5. while(GetMessage){} : 메세지 루프문

```
while (PeekMessage(&msg, handle, 0, 0, PM_REMOVE))
{
    TranslateMessage(&msg);
    DispatchMessage(&msg);
}
```
