---
title: Depth Stencil
author: unwoo52
date: 2022-11-22 00:00:00 +09:00
categories: [DirectX, GameEngine ]
tags: [directX, Win32API, DXGI, GameEngine, Graphics, DepthStencil]
---

## Dex(깊이)버퍼와 Stencill(틀판)버퍼

깊이버퍼 : NDC(3D)공간을 2D로 압축하면서 한 픽셀에는 압축된 3D의 깊이(z축)이 내포되어 있음. 이에 대한 깊이를 뜻하는 덱스에 대한 정보가 들어있는 버퍼

스텐실버퍼 : 원하는 조건에 부합하는 픽셀들만 출력하기 위한 버퍼. 깊이 버퍼와 함께 사용하여 깊이버퍼 내의 원하는 픽셀들만 출력하도록 할 수 있음. 투명하지 않은 벽이라면 가장 앞의 벽만 출력, 투명한 유리라면 유리 + 뒤의 깊이에 있는 픽셀들도 출력해야 최적화에 유리할 것이다. 왜냐하면 뒤가 보이지 않는 벽인데도 뒤의 낮은깊이버퍼들도 렌더링 하는 것은 손해이기 때문.


## Depth Stencil 만들기

필요한 필드 

- Graphics.h

```cpp
	//Depth Stencil
	ComPtr<ID3D11DepthStencilView> dsView;
	ComPtr<ID3D11Texture2D> dsBuffer;
	ComPtr<ID3D11DepthStencilState> ds;
```

Stencil View, Stencil Buffer를 위한 TEXTURE2D, 그리고 Stencil State 3개이다.

----

- Graphics.cpp

```cpp
//Create Depth Stencil State
    //깊이 버퍼를 사용하면 그리는 순서에 관계 없이 이 값에 따라 렌더링 순서가 정해진다.
    D3D11_DEPTH_STENCIL_DESC dsDesc = { 0 };
    dsDesc.DepthEnable = true;
    //깊이를 사용한다D3D11_DEPTH_WRITE_MASK_ALL or 사용안한다 옵션D3D11_DEPTH_WRITE_MASK_ZERO
    dsDesc.DepthWriteMask = D3D11_DEPTH_WRITE_MASK::D3D11_DEPTH_WRITE_MASK_ALL;
    //비교 옵션.
    dsDesc.DepthFunc = D3D11_COMPARISON_FUNC::D3D11_COMPARISON_LESS_EQUAL;
    hr = device->CreateDepthStencilState(&dsDesc, ds.GetAddressOf());
```

먼저 Stencil State 생성 후 정의. 옵션과 사용여부를 정의한다.


```cpp
//Create Depth Stencil Buffer
    D3D11_TEXTURE2D_DESC dsbDesc = { 0 };
    dsbDesc.Width = w;
    dsbDesc.Height = h;
    dsbDesc.MipLevels = 1;
    dsbDesc.ArraySize = 1;
    dsbDesc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;
    dsbDesc.SampleDesc.Count = 1;
    dsbDesc.SampleDesc.Quality = 0;
    dsbDesc.Usage = D3D11_USAGE_DEFAULT;
    dsbDesc.BindFlags = D3D11_BIND_DEPTH_STENCIL;
    dsbDesc.CPUAccessFlags = 0;
    dsbDesc.MiscFlags = 0;
    hr = device->CreateTexture2D(&dsbDesc, NULL, dsBuffer.GetAddressOf());
```

그리고 Stencil Buffer를 위한 TEXTURE2D 정의 및 생성



```cpp
hr = device->CreateDepthStencilView(dsBuffer.Get(), NULL, dsView.GetAddressOf());
```

생성한 Stencil Buffer(TEXTURE2D)와 Stencil State를 사용해 Stencil View 생성


## Depth Stencil 사용하기


### 아웃풋머저 인자 수정하기

++     DC->OMSetRenderTargets(1, renderTargetView.GetAddressOf(), dsView.Get());
 
```cpp
//원래 코드
DC->OMSetRenderTargets(1, renderTargetView.GetAddressOf(), NULL);

//수정 후 코드     
DC->OMSetRenderTargets(1, renderTargetView.GetAddressOf(), dsView.Get());
```

### 렌더링 할 때 마다 CLEAN 실행하기

```cpp
void Graphics::RenderFrame()
{
    DC->ClearRenderTargetView(renderTargetView.Get(), bgcolor);
    DC->ClearDepthStencilView(dsView.Get(), D3D11_CLEAR_DEPTH | D3D11_CLEAR_STENCIL, 1, 0);
}
```

백버퍼를 청소할 때, DepthStencil도 같이 청소한다.

