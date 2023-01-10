---
title: constant buffer
author: unwoo52
date: 2022-11-24 10:00:00 +09:00
categories: [DirectX, GameEngine ]
tags: [directX, Win32API, DXGI, GameEngine, Graphics, ConstantBuffer, 3DLotate, 3D]
---


## 개요

이미지를 움직여보자

그림의 위치를 결정해주는 변수는 버텍스 셰이더에 있음

## 버텍스셰이더에 위치값 멤버 생성

![imagename](/assets/image/DirectX/GameEngine/Rendering-Pipeline/001.png)

메모리리소스영역에 보면, 작게 콘스탄트버버도 포함되어있는 것을 볼 수 있다.

메모리 소스 영역 : 앱과 렌더링파이프라인 양쪽에서 접근할 수 있는 공유되는 메모리 영역이다. 여기에 위치값을 두고 사용하면 빠르게 그림의 위치를 실시간으로 바뀌게 할 수 있음.

- vs.hlsh

```hlsl
cbuffer buffer : register(b0)
{
    float2 Offset;
}
```

버텍스셰이더에 cbuffer로 필드를 추가.

cbuffer를 무한히 많이 만들수는 없다. 갯수제한이 있음.

Offset : 보정을 위한 멤버. 아래에서 어떻게 사용하는지 설명함.

------

- Buffer.h

```hlsl
PS_INPUT vs(VS_INPUT inPut)
{
    PS_INPUT outPut;
    //위치에 보정을 한 모습
    inPut.pos.x += Offset.x;
    inPut.pos.y += Offset.y;
    
    outPut.pos = float4(inPut.pos, 1.0f);
    outPut.color = inPut.color;
    outPut.uv = inPut.uv;
	return outPut;
}
```


## ConstantBufferTypes.h

```cpp
#pragma once
#include <DirectXMath.h>
using namespace DirectX;

struct CB_VS_Offset
{
	XMFLOAT2 offset;
};

struct CB_VS_Transform
{
	XMMATRIX mat;
};

struct CB_PS_Alpha
{
	float alpha;
};
```

평면위치(offset)값과, 3D상에서의 위치(transform)값에 대한 행렬(Matrix)과, 투명도를 위한 alpha값을 갖고 있다. 이 구조체들을 ConstantBuffer\<T\>에서 사용하게 된다.

------

## ConstantBuffer Class 생성

- Buffers.h


```cpp
//버퍼 클래스
class Buffer
{
protected:
	ComPtr<ID3D11Buffer> buffer;
	UINT size = 0;
public:
	...
};

template<class T>
class ConstantBuffer : public Buffer
{
	ID3D11DeviceContext* dc = nullptr;
public:
	T data;
	ConstantBuffer() {}

	HRESULT Initialize(ID3D11Device* device, ID3D11DeviceContext* dc)
	{
		if (buffer.Get() != nullptr) buffer.Reset();

		this->dc = dc;

		//Create Constant Buffer
		D3D11_BUFFER_DESC cbDesc = { 0 };
		//계속 값이 바뀌기 때문에 동적
		cbDesc.Usage = D3D11_USAGE_DYNAMIC;
		cbDesc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
		//우리쪽에서도 WRITE할 수 있게 하는 옵션
		cbDesc.CPUAccessFlags = D3D10_CPU_ACCESS_WRITE;
		cbDesc.MiscFlags = 0;
		//Constant Buffer의 크기는 16의 배수로 만들어져야 하기 때문에 추가 연산
		cbDesc.ByteWidth = sizeof(T) + (16 - sizeof(T) % 16);
		cbDesc.StructureByteStride = 0;

		return device->CreateBuffer(&cbDesc, 0, buffer.GetAddressOf());
	}

    bool Update()
    {
    	//데드락 막기
        //락 선언
        D3D11_MAPPED_SUBRESOURCE map;

        //콘스탄스버퍼의 데이터를 락을 걸고 데이터를 카피해서 가져옴 > &map에 주소가 담김
        HRESULT hr = dc->Map(buffer.Get(), 0, D3D11_MAP_WRITE_DISCARD, 0, &map);

        if (FAILED(hr))
            fail code...

        //pData : 이식하고자 하는 데이터. 이를 CopyMemory()를 이용해 아까 &map로 구한 주소 위치에 값을 대입
        CopyMemory(map.pData, &data, sizeof(T));

        dc->Unmap(buffer.Get(), 0);
        return true;
    }
};

VertexBuffer...

IndexBuffer...
```

[전체 코드](https://unwoo52.github.io/posts/Buffers/)

c++은 템플릿을 쓰기 위해 헤더파일에서 선언과 정의를 동시에 해야 한다. 지금 만들 buffers는 헤더파일만 있고 cpp파일은 없음.

-----------

- Graphics.h

```cpp
public:
	ConstantBuffer<CB_VS_Transform> cbvs;
	ConstantBuffer<CB_PS_Alpha> cbps;
```

- Graphics::InitializeScene()

```cpp
    hr = cbvs.Initialize(device.Get(), DC.Get());
    if (FAILED(hr))
		...
        
    //XMMATRIX world를 단위행렬값으로 초기화.
    world = XMMatrixIdentity();

    hr = cbps.Initialize(device.Get(), DC.Get());
		...
```




### 렌더링 실행

- Graphics::RenderFrame()

```cpp
	//cbvs 행렬값에 XMMatrixTranspose연산을 수행
    cbvs.data.mat = XMMatrixTranspose(world * camera.GetViewProjMatrix());
    //Update Constant Buffer    
    if (!cbvs.Update()) return;
    DC->VSSetConstantBuffers(0, 1, cbvs.GetAddressOf());

    static float alpha = 1.0f;
    cbps.data.alpha = alpha;
    if (!cbps.Update()) return;
    DC->PSSetConstantBuffers(0, 1, cbps.GetAddressOf());
```


