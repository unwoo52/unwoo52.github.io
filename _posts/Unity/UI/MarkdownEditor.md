---
title: Unity UI 구현하기
author: unwoo52
date: 2022-11-13 00:00:00 +09:00
categories: [Unity, UI]
tags: [Unity, UI]
---

## UI를 구현하는 방법들

- IMGUI

DX9시절에 쓰던 2d로 ui를 구현하는 방법. DX11에 와서는 렌더링파이프라인이 오직 3d만을 위해 구성되었기 때문에 2d를 사용한 IMGUI는 역으로 무겁고 느려지게 되어서 잘 사용하지 않는다.

- NGUI

3d를 2d처럼 보이게 구현. 3d로 돌아가기 때문에 DX11에서도 빠르게 돌아가면서도 2d로 보이게 UI를 보여줌

- UGUI

유니티에서 NGUI의 UI구현방식 라이센스를 사와서 모방해 만든 UI