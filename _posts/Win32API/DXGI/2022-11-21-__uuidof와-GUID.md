---
title: __uuidof와 GUID
author: unwoo52
date: 2022-11-21 00:00:00 +09:00
categories: [Win32API]
tags: [Win32API, GUID]
---

## __uuidof

```cpp
__uuidof (expression)
```

expression에 연결된 GUID를 검색한다.

## __LIBID_

라이브러리 이름이 더 이상 범위에 없는 경우 __LIBID_를 대신 사용할  수 있다.

```cpp
StringFromCLSID(__LIBID_, &lpolestr);
```

## GUID

마이크로소프트 컴포넌트 오브젝트 모델( COM )에서는 GUID를 구성 요소의 인터페이스를 구별하기 위해 사용한다.

이 포스트를 쓰게 된 이유가 되는 [렌더링 파이프라인 구현](https://unwoo52.github.io/posts/Graphics/#graphicsh)에서 COM을 사용하니 이러한 이유로 사용되는 것이라고 알 수 있다.

## 참고

[__uuidof 마이크로소프트 공식 문서](https://learn.microsoft.com/ko-kr/cpp/cpp/uuidof-operator?view=msvc-170)

[GUID 블로그 포스트](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=ljc8808&logNo=220456743162)

[GUID wiki](https://ko.wikipedia.org/wiki/%EC%A0%84%EC%97%AD_%EA%B3%A0%EC%9C%A0_%EC%8B%9D%EB%B3%84%EC%9E%90)
