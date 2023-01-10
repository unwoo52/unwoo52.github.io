---
title: Rasterize
author: unwoo52
date: 2022-11-21 08:00:00 +09:00
categories: [DirectX, GameEngine]
tags: [directX, Win32API, DXGI, GameEngine, Graphics, Rasterize]
---

## Rasterize Setting

- bool Graphics::InitializeDirectX(HWND hWnd, int w, int h)

```cpp
#define DC deviceContext

...

//Create Viewport
D3D11_VIEWPORT vp = { 0 };
vp.TopLeftX = 0;
vp.TopLeftY = 0;
vp.Width = w;
vp.Height = h;
vp.MinDepth = 0;
vp.MaxDepth = 1;
//Set to Rendering pipeline
DC->RSSetViewports(1, &vp);
```

뷰포트의 사이즈를 정의하고, deviceContext의 RS에서 뷰포트 정의
