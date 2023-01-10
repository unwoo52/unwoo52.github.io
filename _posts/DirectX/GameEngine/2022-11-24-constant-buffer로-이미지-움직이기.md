---
title: constant buffer로 이미지 움직이기
author: unwoo52
date: 2022-11-24 10:00:00 +09:00
categories: [DirectX, GameEngine ]
tags: [directX, Win32API, DXGI, GameEngine, Graphics]
---


## 개요

cbuffer값을 추가하고, 렌더링할 때 마다 적용하게 하여 이미지를 움직이는 기능을 구현해보자.

cbuffer는 Memort Resource영역에 저장되어 있기 때문에 렌더링 파이프라인과 앱 양쪽에서 빠르게 접근이 가능해서 매 프레임마다 갱신되는 offset(위치를 보정할 값)을 처리하기에 알맞지만, 데드락현상이나 기타 자잘하게 발생할 예쌍치 못한 오류가 발생할 수 있기 때문에 이를 막기 위해서 map을 이용한 락도 사용해보겠다.

## 버텍스셰이더에 위치값 멤버 생성

![imagename](/assets/image/DirectX/GameEngine/Rendering-Pipeline/001.png)

메모리리소스영역에 보면, 작은 글씨로 constant buffer도 포함되어있는 것을 볼 수 있다.

메모리 소스 영역 : 앱과 렌더링파이프라인 양쪽에서 접근할 수 있는 공유되는 메모리 영역이다. 여기에 constant buffer를 두고 사용하면 앱과 렌더링파이프라인에서 빠르게 접근이 가능하다는 장점이 있음.

### 버텍스셰이더에 

- vs.hlsh

```hlsl
cbuffer buffer : register(b0)
{
    float2 Offset;
}

...

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

버텍스셰이더에 cbuffer로 필드를 추가.

cbuffer를 무한히 많이 만들수는 없다. 갯수제한이 있음.

Offset : 위치값 보정을 위한 멤버.


------

## Offset 구조 선언

- ConstantBufferTypes.h

```cpp
#pragma once
#include <DirectXMath.h>
using namespace DirectX;

struct CB_VS_Offset
{
	XMFLOAT2 offset;
};
```

평면위치(offset)값을 갖고 있다. 이 구조체들을 ConstantBuffer\<T\>에서 사용하게 된다.

------

## 버퍼 생성

Buffers.h에 ConstantBuffer Class 생성

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
        //메모리 담을 공간 선언
        D3D11_MAPPED_SUBRESOURCE map;

        //메모리리소스에서 cbuffer에 락을 걸고, cbuffer를 가져와 map에 담음
        HRESULT hr = dc->Map(buffer.Get(), 0, D3D11_MAP_WRITE_DISCARD, 0, &map);

        if (FAILED(hr))
            fail code...

        //map의 pData값을 수정된 data로 갱신
        CopyMemory(map.pData, &data, sizeof(T));

        //언락
        dc->Unmap(buffer.Get(), 0);
        return true;
    }
};

VertexBuffer {...}

IndexBuffer {...}
```

[전체 코드](https://unwoo52.github.io/posts/Buffers/)

c++은 템플릿을 쓰기 위해 헤더파일에서 선언과 정의를 동시에 해야 한다. 지금 만들 buffers는 헤더파일만 있고 cpp파일은 없음.

-----------

## cbuffer 사용하기

### 버퍼 정의

- Graphics.h

```cpp
public:
	ConstantBuffer<CB_VS_Offset> cb;
```

- Graphics::InitializeScene()

```cpp
    hr = cb.Initialize(device.Get(), DC.Get());
    if (FAILED(hr))
        fail code...
```

### 키보드 입력을 통해  cbuffer값 바꾸기

<details>

  - Engine::Update()
  
  ```cpp
  while (!keyboard.KeyBufferIsEmpty())
  {
      KeyboardEvent ke = keyboard.ReadKey();
      unsigned char key = ke.GetKeyCode();
      if (key == VK_LEFT)
      {
          gfx.cb.data.offset.x -= 0.1f;
      }
      if (key == VK_RIGHT)
      {
          gfx.cb.data.offset.x += 0.1f;
      }
  }
  ```
</details>

### 렌더링 실행

- Graphics::RenderFrame()

```cpp
    //TEST CODE
    cb.data.offset.x = -0.5f;
    cb.data.offset.y = 0.0f; 
    //Update Constant Buffer    
    if (!cb.Update()) return;

    //메모리소스영역에 다시 ConstantBuffer 갱신
    DC->VSSetConstantBuffers(0, 1, cb.GetAddressOf());
```

키입력을 통해 바뀐 cbuffer값이 반영되어 좌우키를 통해 사진이 움직이게 된다.
