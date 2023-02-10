---
title: DirectX로 GameEngine 만들기 프로젝트 개요
author: unwoo52
date: 2022-12-11 00:00:00 +09:00
categories: [DirectX, GameEngine]
tags: [directX, Win32API, GameEngine]
---

## 프로젝트 개요

이번 프로젝트에서는 Win32API와 DirectX, DXGI를 이용하여 직접 게임 엔진을 구현해보고, 게임 엔진이 실제로 어떻게 마우스, 키보드 등의 입력을 받고 처리하는지, 렌더링파이프라인을 사용해보고 그래픽이 어떻게 화면에 렌더링 되는지를 배워보자 한다.

## 개발

### DirectXTK 설정

DirectX와 DXGI를 사용하기 위해서 DirecXTK를 microsoft공식 사이트에서 다운로드 받아 빌드하고 프로젝트 안에 포함시켜야 한다.

[DirectXTK](https://unwoo52.github.io/posts/AdapterReader/)

----

### 윈도우 생성하고 관리하기

윈도우를 생성하고, 어플리케이션 상태 관리를 수행한다.

[윈도우 생성 및 애플리케이션 상태 관리](https://unwoo52.github.io/posts/%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EC%83%81%ED%83%9C-%EA%B4%80%EB%A6%AC/)

----

### 마우스 및 키보드 입력 처리하기

이벤트 버퍼를 이용해 키와 마우스의 입력을 처리하는 기능을 구현한다.

[Keyboard 입력 구현](https://unwoo52.github.io/posts/Key-%EC%9E%85%EB%A0%A5-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0/)

[Mouse 입력 구현](https://unwoo52.github.io/posts/Mouse-%EC%9E%85%EB%A0%A5-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0/)

----

#### 그래픽 구현하기

DXGI를 사용해 그래픽카드에 그림을 구현해보고 스왑체인을 이용해 화면에 출력하는 기능을 구현해본다.


