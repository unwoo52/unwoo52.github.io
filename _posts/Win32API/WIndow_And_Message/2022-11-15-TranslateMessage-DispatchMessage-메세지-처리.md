---
title: TranslateMessage DispatchMessage 메세지 처리
author: unwoo52
date: 2022-11-15 00:00:00 +09:00
categories: [Win32API]
tags: [Win32API]
---


GetMessage 혹은 PeekMessage를 통해 메세지를 받았지만 이 구조를 직접 검사하지는 않는다. 윈도우는 메세지를 TranslateMessage와 DispatchMessage에 전달해 처리하게 한다.

## TranslateMessage

반드시 DispatchMessage이전에 호출되어야 한다.

우리가 키보드를 통해 한 문자열 ex)a를 눌렀을 때 일어나는 처리를 알아보자.

a버튼을 눌러 메세지를 넘기면 WM_KEYDOWN, WM_CHAR, WM_KEYUP이 연속적으로 발생한다.

근데 이중에 WM_CHAR는 사용자에 의해 발생한 메세지가 아니라 **TranslateMessage**에서 발생한다.

TranslateMessage는 지금 입력된 키가 문자키인지 아닌지(마우스와 같은) 판단한 후 문자열일 경우 WM_CHAR를 생성하는 것이다.

TranslateMessage를 주석으로 가리고 실행해보면 문자열 입력이 먹히지 않는 것을 구경할 수 있다.

## DispatchMessage

메세지를 메세지프로시저로 전달해준다. 메세지프로시저 포스트에서 계속
