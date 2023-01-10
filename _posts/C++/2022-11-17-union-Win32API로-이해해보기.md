---
title: Union, Win32API로 이해해보기
author: unwoo52
date: 2022-11-17 00:00:00 +09:00
categories: [C++]
tags: [C++, directX, Win32API, GameEngine, MouseInput, InputProcess, union, short, WinUser]
---


## 개요

union의 기능은 이미 알고있었지만게임 엔진을 구현하면서 실제로 어떻게 동작하는지를 볼 수 있는 기회를 얻게 되었다.

```cpp
unique_ptr<BYTE[]> rawdata = make_unique<BYTE[]>(size);			
if (GetRawInputData(reinterpret_cast<HRAWINPUT>(lParam), RID_INPUT, rawdata.get(), &size, sizeof(RAWINPUTHEADER)) == size)
{
	RAWINPUT* raw = reinterpret_cast<RAWINPUT*>(rawdata.get());
	if (raw->header.dwType == RIM_TYPEMOUSE)
	{
		mouse.OnMouseMoveRaw(raw->data.mouse.lLastX, raw->data.mouse.lLastY);
	}
}
```

게임 엔진에서 마우스 인풋 구현과 MouseRaw(마우스가 이동한 거리 값)을 구하는 과정에서 union을 파헤칠 기회를 얻었다.

위의 코드에서 RAWINPUT에 대해 살펴보도록 하겠다.

----------

**윈도우 내부가 어떻게 생겼는지를 파헤치는 것은 언제나 즐거운 일이다.**

**c++ 공부에 대한 내용을 개발에서 깨우치게 되는 것은 더더욱 즐거운 일이다.**

**이 두가지를 같이 한다고? 오, 나 혼절**

**포스팅 하는 것을 참을수가 없었다. 바로 시작.**

## Union

구조체와 유사하지만 데이터를 저장하는 방법에 있어서 차이를 갖는다.

## 공용체와 구조체의 차이

![imagename](/assets/image/C++/union/001.png)

```cpp
struct myStruct{
	char c;
    short s;
    int i;
    }
```

```cpp
union myUnion{
	char c;
    short s;
    int i;
}
```

구조체는 일반적으로 우리가 상상할 수 있는 방법으로 데이터가 저장된다.

하지만 공용체는 다르다. 공용체 내의 멤버중 가장 크기가 큰 멤버 변수의 크기로 메모리를 할당받고, 나머지 작은 멤버들은 가장 큰 멤버의 메모리를 공유한다.

## 공용체 사용해보기

```cpp
union union_t{
	int   num;
    char  a;
    char  b;
    char  c;
    char  d;
    }
```

![imagename](/assets/image/C++/union/002.png)

다음과 같은 형태로 저장이 된다.

만약 abcd를 union_t의 4개의 바이트 각각에 하나씩 저장되게 하고 싶으면?

구조체와 같이 사용하면 된다.

```cpp
union union_t{
   int     num;
   struct {
      char  a;
      char  b;
      char  c;
      char  d;
   };
}; 
```

![imagename](/assets/image/C++/union/003.png)

## RAWINPUT의 구조

다시 처음으로 돌아와서 RAWINPUT을 살펴보자. RAWINPUT을 F12로 접근하면 다음과 같은 형태를 볼 수 있다.

```cpp
typedef struct tagRAWINPUT {
    RAWINPUTHEADER header;
    union {
        RAWMOUSE    mouse;
        RAWKEYBOARD keyboard;
        RAWHID      hid;
    } data;
} RAWINPUT, *PRAWINPUT, *LPRAWINPUT;
```

tagRAWINPUT구조체 안에 header와 data가 있다.


## 옆길로 세어버리기

### header

```cpp
typedef struct tagRAWINPUTHEADER {
    DWORD dwType;
    DWORD dwSize;
    HANDLE hDevice;
    WPARAM wParam;
} RAWINPUTHEADER, *PRAWINPUTHEADER, *LPRAWINPUTHEADER;
```

각각 long, long, void\*, uint_ptr 형태이다.

궁금증이 생겨서 uint_ptr인 wParam에도 접근해보았다.

- wParam

```cpp
typedef UINT_PTR            WPARAM;
```

- UINT_PTR

```cpp
#if defined(_WIN64)
    typedef __int64 INT_PTR, *PINT_PTR;
    typedef unsigned __int64 UINT_PTR, *PUINT_PTR;

    typedef __int64 LONG_PTR, *PLONG_PTR;
    typedef unsigned __int64 ULONG_PTR, *PULONG_PTR;

    #define __int3264   __int64

#else
    typedef _W64 int INT_PTR, *PINT_PTR;
    typedef _W64 unsigned int UINT_PTR, *PUINT_PTR;

    typedef _W64 long LONG_PTR, *PLONG_PTR;
    typedef _W64 unsigned long ULONG_PTR, *PULONG_PTR;

    #define __int3264   __int32

#endif
```

왜 그냥 사용하지 않고 define을 이용해서 다른 이름으로 사용하는지 궁금했는데, \#if defined(\_WIN64)을 보고 이해했다. 각기 다른 환경에서 실행될 때 마다 다른 정의가 필요해서였다... \_WIN64환경에선 \_\_int64으로 정의되고 이 외의 환경(아마 32)에서는 \_W64로 정의되는 것 이다.

### w64

궁금해서 w64에 대해서도 검색해서 마이크로소프트 문서를 읽어보았는데

- 이 Microsoft 관련 키워드는 더 이상 사용되지 않습니다. Visual Studio 2013 이전 버전에서는 변수를 표시할 수 있으므로 /Wp64로 컴파일할 때 64비트 컴파일러로 컴파일할 때 보고되는 경고를 컴파일러가 보고합니다.

라는 내용이다.

[\_w64에 대한 내용](https://unsealedwiener.blogspot.com/2019/10/w64.html)

무시해도 될 것 같은 키워드이다.

헤더파일에 접근해 전처리기들을 읽는것이 꽤 재미있다. 내친김에 다른 것들도 구경해보자.

### HANDLE

- HANDLE

```cpp
#ifdef STRICT
typedef void *HANDLE;
#if 0 && (_MSC_VER > 1000)
#define DECLARE_HANDLE(name) struct name##__; typedef struct name##__ *name
#else
#define DECLARE_HANDLE(name) struct name##__{int unused;}; typedef struct name##__ *name
#endif
```

조건으로 STRICT이 있다. 무슨 조건일까? 한번 접근해 보았다.

### STRICT

- minwindef.h

```cpp
#pragma region Application Family or OneCore Family or Games Family
#if WINAPI_FAMILY_PARTITION(WINAPI_PARTITION_APP | WINAPI_PARTITION_SYSTEM | WINAPI_PARTITION_GAMES)

#ifndef NO_STRICT
#ifndef STRICT
#define STRICT 1
#endif
#endif /* NO_STRICT */

// Win32 defines _WIN32 automatically,
// but Macintosh doesn't, so if we are using
// Win32 Functions, we must do it here
```

정확하게는 모르겠어도 STRICT가 항상 1(true)를 정의한다고 생각할 수 있을 것 같다. 이변이 없는 한 true로 생각하면 될 것 같다.

### 매킨토시

위의 minwindef.h에서 본 STRICT 밑의 주석문이 궁금해서 번역기를 돌려봤다.

- Win32는 _WIN32를 자동으로 정의하지만 Macintosh는 그렇지 않기 때문에 Win32 함수를 사용하는 경우 여기서 해야 합니다.

세상에 매킨토시라니 익숙한 이름이다! 분명 굉장히 과거의 컴퓨터였는데 정확하게 그게 뭐였지? 당장 검색해보았다.

![imagename](/assets/image/C++/union/004.png)

1학년 때 컴퓨터과학 수업에서 컴퓨터의 역사 항목에서 보던 컴퓨터였다. 마치 고고학처럼 오래된 유물을 파해치는 기분이다. 아마 우리 주변의 소프트웨어를 이루는 근간 이 되는 다른 코드들에 접근하면 이런 주석들이 더 많지 않을까? 깊숙히 숨은 다른 코드들도 궁금해진다.

### 전처리기

다시 minwindef.h로 돌아와서 코드를 구경한다. 전에 공부는 했지만 잘 이해하지 못하고 지나갔던 전처리기들이 궁금해졌다. 다시 또 전처리기로 빠져서 공부해본다...

[전처리기 링크](https://blog.naver.com/sharonichoya/220507818075)

무슨 나무위키 보는 기분이다. 나는 3학년 때 시험 일주일전에 공부하다 말고 갑자기 무슨 바람이 분건지 아폴로미션을 정주행 하느라 그 날 저녁을 날린적이 잇다.

그날은 결국 모든 미션과 로켓, 착륙선을 구경하고도 모자라서 21세기의 우주탐사미션에 대한 문서들을 죄다 읽고 새벽이 되어서야 잠들고 말았다.

![imagename](/assets/image/C++/union/005.png)

지금 코드를 탐색하는 것이 나무위키를 읽는 기분이다. 여길 읽으면 저기도 읽게되고, 링크에 링크를 타고 새로운걸 배우러 가고...


## RAWINPUT의 공용체 멤버들

흥분을 가라앉히고 다시 원래의 본점으로 돌아오자. 분명 처음에는 오늘 배운 마우스 인풋에 대해 포스트를 작성하려고 했는데 무슨 일기처럼 되어버렸다. 다음엔 아에 따로 일기 항목을 개설할까보다.

```cpp
typedef struct tagRAWINPUT {
    RAWINPUTHEADER header;
    union {
        RAWMOUSE    mouse;
        RAWKEYBOARD keyboard;
        RAWHID      hid;
    } data;
} RAWINPUT, *PRAWINPUT, *LPRAWINPUT;
```

RAWINPUT으로 돌아왔다. 마지막 줄의 다른 별칭들을 보면,, 검색 안해봐도 뻔하다. 포인터RAWINPUT, 롱포인터RAWINPUT일 것 같다. 

아무튼 RAWINPUT(data) 공용체는 마우스 키보드 아이디 값을 갖고 있다. 각각의 멤버도 어떻게 생겼는지 궁금하니 접근해보자.

### RAWMOUSE

```cpp
typedef struct tagRAWMOUSE {
    /*
     * Indicator flags.
     */
    USHORT usFlags;

    /*
     * The transition state of the mouse buttons.
     */
    union {
        ULONG ulButtons;
        struct  {
            USHORT  usButtonFlags;
            USHORT  usButtonData;
        } DUMMYSTRUCTNAME;
    } DUMMYUNIONNAME;


    /*
     * The raw state of the mouse buttons.
     */
    ULONG ulRawButtons;

    /*
     * The signed relative or absolute motion in the X direction.
     */
    LONG lLastX;

    /*
     * The signed relative or absolute motion in the Y direction.
     */
    LONG lLastY;

    /*
     * Device-specific additional information for the event.
     */
    ULONG ulExtraInformation;

} RAWMOUSE, *PRAWMOUSE, *LPRAWMOUSE;
```

long 4개, short 플래그 한개, 공용체 하나가 있다. 공용체도 long(4bytes)형 크기에 short(2bytes)가 두개 들어가있다.

data 공용체의 세 멤버중 가장 크다. 이 크기를 기준으로 공용체가 메모리에 할당될 것이다.

### RAWKEYBOARD

```cpp
typedef struct tagRAWKEYBOARD {
    /*
     * The "make" scan code (key depression).
     */
    USHORT MakeCode;

    /*
     * The flags field indicates a "break" (key release) and other
     * miscellaneous scan code information defined in ntddkbd.h.
     */
    USHORT Flags;

    USHORT Reserved;

    /*
     * Windows message compatible information
     */
    USHORT VKey;
    UINT   Message;

    /*
     * Device-specific additional information for the event.
     */
    ULONG ExtraInformation;


} RAWKEYBOARD, *PRAWKEYBOARD, *LPRAWKEYBOARD;
```

short 4개, int 한개, long 한개

### RAWHID

ID 핸들값

```cpp
typedef struct tagRAWHID {
    DWORD dwSizeHid;    // byte size of each report
    DWORD dwCount;      // number of input packed
    BYTE bRawData[1];
} RAWHID, *PRAWHID, *LPRAWHID;
```

DWORD는 long, BYTE는 char 크기이다. long2개, char1개. 가장 작다.

### 결론

시스템 입장에서는 공용체가 편하고 송수신하기에 좋다. 공용체에 들어가는 세 멤버의 형태와 역할, 각자 크기는 달라도 같은 크기와 형태의 공용체로 선언되어 있기 때문에 세 경우마다 다른 값을 받거나 각기의 액션을 할 필요가 없다. RAWINPUT이라는 값으로만 받기 때문에 편할 것 이다.

개발자 또한 공용체가 직관적이고 이해하기도 쉽다. RAWINPUT 하나를 사용하면서도 mouse, keyboard, hid가 들어있다는 사실을 직접  볼 수 있기 때문이다.

이제 공용체가 왜 송수신하는 데이터 형식으로 그렇게 많이 보이는지 잘 알게 되었다. 게임 엔진에서 실제로 RAWINPUT을 어떻게 처리했는지 다시 확인해보고 마무리하겠다.

```cpp
if (raw->header.dwType == RIM_TYPEMOUSE)
{
	mouse.OnMouseMoveRaw(raw->data.mouse.lLastX, raw->data.mouse.lLastY);
}
```

RAWINPUT을 raw로 받은 뒤, header의 타입을 조사해 마우스인지 확인하는 조건을 통해 값을 처리하는 모습이다. 이제 공용체가 어떻게 사용되는지, 또 Win32API에서 인풋을 어떤 모양으로 송수신하고 처리하는지를 이번 경험을 통해 더 구체적으로 알게 되었다.

## 참고

[TCP 코딩의 시작](http://www.tcpschool.com/c/c_struct_unionEnum)

[블로그](https://badayak.com/entry/C%EC%96%B8%EC%96%B4-%EA%B3%B5%EC%9A%A9%EC%B2%B4-union-%EC%89%AC%EC%9A%B4-%EC%84%A4%EB%AA%85)

[마이크로소프트 w64](https://learn.microsoft.com/ko-kr/cpp/cpp/w64?view=msvc-170)

