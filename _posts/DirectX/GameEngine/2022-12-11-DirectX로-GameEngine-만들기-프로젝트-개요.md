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

### 그래픽 구현하기

DXGI를 사용해 그래픽카드에 그림을 구현해보고 스왑체인을 이용해 화면에 출력하는 기능을 구현해본다.

[어뎁터 리더 구현](https://unwoo52.github.io/posts/AdapterReader/)

[Graphics Class 구현](https://unwoo52.github.io/posts/Graphics/)

렌더링 파이프라인을 이해하고 사용해본다

[렌더링 파이프라인](https://unwoo52.github.io/posts/Rendering-Pipeline/)

[버텍스 셰이더와 픽셸 셰이더](https://unwoo52.github.io/posts/Vertex-Shader-and-Pixel-Shader/)

[삼각형 그려보기](https://unwoo52.github.io/posts/%EC%82%BC%EA%B0%81%ED%98%95-%EA%B7%B8%EB%A0%A4%EB%B3%B4%EA%B8%B0/)

[레스트라이저](https://unwoo52.github.io/posts/Rasterize/)

[깊이버퍼와 스텐실버퍼](https://unwoo52.github.io/posts/Depth-Stencil/)

[프로젝트에 폰트 적용하고 글씨 그려보기](https://unwoo52.github.io/posts/Font/)

[윈도우 크기 맞추기](https://unwoo52.github.io/posts/window-%ED%81%AC%EA%B8%B0-%EC%A0%95%ED%99%95%ED%95%98%EA%B2%8C-%EB%A7%8C%EB%93%A4%EA%B8%B0/)

[인덱스버퍼](https://unwoo52.github.io/posts/IndexBuffer/)

[uv로 도형에 이미지 그리기](https://unwoo52.github.io/posts/uv%EB%A1%9C-%EB%8F%84%ED%98%95%EC%97%90-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EA%B7%B8%EB%A6%AC%EA%B8%B0/)

[constant buffer](https://unwoo52.github.io/posts/constant-buffer%EB%A1%9C-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%9B%80%EC%A7%81%EC%9D%B4%EA%B8%B0/)

### 선형대수학을 이용해 3d 화면 구현하기

[행렬](https://unwoo52.github.io/posts/%ED%96%89%EB%A0%AC/)

[카메라](https://unwoo52.github.io/posts/%EC%B9%B4%EB%A9%94%EB%9D%BC/)

[3d물체 움직이기](https://unwoo52.github.io/posts/3d-%EC%A2%8C%ED%91%9C%EC%97%90%EC%84%9C-%EC%9B%80%EC%A7%81%EC%9D%B4%EA%B8%B0/)


