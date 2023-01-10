---
title: IndexBuffer
author: unwoo52
date: 2022-11-24 05:00:00 +09:00
categories: [DirectX, GameEngine ]
tags: [directX, Win32API, DXGI, GameEngine, Graphics, IndexBuffer]
---


## 인덱스버퍼 개요

 버텍스버퍼로 사용하면 하나의 그래픽을 그리기 위한 정보(data)의 크기가 쓸대없이 커지게 됨.
 
 인덱스로 데이터를 저장하고 사용하면 더 적은 데이타를 가지면서도 같은 그림을 그릴 수 있게 됌
 
## 인덱스 버퍼

- Buffers.h

```cpp
class IndexBuffer : public Buffer
{	
public:
	IndexBuffer() {};	

	HRESULT Initialize(ID3D11Device* device, DWORD* data, UINT numIndices)
	{
		size = numIndices;

		D3D11_BUFFER_DESC ibDesc = { 0 };
		ibDesc.Usage = D3D11_USAGE_DEFAULT;
		ibDesc.ByteWidth = sizeof(DWORD) * numIndices;
		ibDesc.BindFlags = D3D11_BIND_INDEX_BUFFER;
		ibDesc.CPUAccessFlags = 0;
		ibDesc.MiscFlags = 0;

		D3D11_SUBRESOURCE_DATA ibData = { 0 };
		ibData.pSysMem = data;

		return device->CreateBuffer(&ibDesc, &ibData, buffer.GetAddressOf());		
	}
};
```
 
## 버퍼 선언

- Graphics.cpp

```cpp
	VertexBuffer<Vertex> vb;
	IndexBuffer ib;
```

## InitializeScene()

- bool Graphics::InitializeScene()

- 기존의 버텍스버퍼 크기

```cpp
    Vertex v[] =
    {
        Vertex(-0.5f, 0.5f, 0.0f, XMFLOAT3(1,1,1),{0,0}),//0
        Vertex( 0.5f, 0.5f, 0.0f, XMFLOAT3(1,1,1), {2,0}),//1
        Vertex(-0.5f, -0.5f, 0.0f, XMFLOAT3(1,1,1), {0,2}),//2

        Vertex(0.5f, 0.5f, 0.0f, XMFLOAT3(1,1,1), {2,0}),//1
        Vertex( 0.5f, -0.5f, 0.0f, XMFLOAT3(1,1,1), {2,2}),//3
        Vertex(-0.5f, -0.5f, 0.0f, XMFLOAT3(1,1,1), {0,2}),//2
    };
```

----
- 새로 그린 버텍스 버퍼 (인덱스버퍼와 함께 쓰기 때문에 6줄>4줄로 줄어들음)

```cpp
    Vertex v[] =
    {
        Vertex(-0.5f, 0.5f, 0.0f, XMFLOAT3(1,1,1),{0,0}),//0
        Vertex( 0.5f, 0.5f, 0.0f, XMFLOAT3(1,1,1), {1,0}),//1
        Vertex(-0.5f, -0.5f, 0.0f, XMFLOAT3(1,1,1), {0,1}),//2        
        Vertex( 0.5f, -0.5f, 0.0f, XMFLOAT3(1,1,1), {1,1}),//3        
    };
```

- 인덱스버퍼 새로 만들기

```cpp
    DWORD indices[] =
    {
        0, 1, 2,
        1, 3, 2
    };     
```

## 그리기

- void Graphics::RenderFrame()

- 기존 코드

```cpp
    //square
    DC->PSSetShaderResources(0, 1, tex.GetAddressOf());
    DC->PSSetSamplers(0, 1, samp.GetAddressOf());
    DC->IASetVertexBuffers(0, 1, square.GetAddressOf(), StridePtr(), &offset);
    DC->Draw(6, 0);
```

- 수정한 코드

```cpp
    //square
    DC->PSSetShaderResources(0, 1, tex.GetAddressOf());
    DC->PSSetSamplers(0, 1, samp.GetAddressOf());
    DC->IASetVertexBuffers(0, 1, vb.GetAddressOf(), vb.StridePtr(), &offset);
    DC->IASetIndexBuffer(ib.Get(), DXGI_FORMAT_R32_UINT, 0);
    DC->DrawIndexed(6, 0, 0);
```

IASetIndexBuffer을 추가하고, Draw대신 DrawIndexed를 사용하여 그렸다.


## Buffers 제네릭화


### 템플릿 사용해서 버텍스 버퍼 개선하기

- c++은 템플릿을 쓰기 위해 헤더파일에서 선언과 정의를 동시에 해야 한다. 지금 만들 buffers는 헤더파일만 있고 cpp파일은 없음

- Buffers.h

```cpp
template<class T>
class VertexBuffer : public Buffer
{	
	UINT stride = 0;	
public:
	VertexBuffer() {};	

	const UINT Stride() const
	{
		return stride;
	}
	
	const UINT* StridePtr() const
	{
		return &stride;
	}

	HRESULT Initialize(ID3D11Device* device, T* data, UINT numVertices)
	{
		size = numVertices;
		stride = sizeof(T);

		D3D11_BUFFER_DESC vbDesc = { 0 };
		vbDesc.Usage = D3D11_USAGE_DEFAULT;
		vbDesc.ByteWidth = sizeof(T) * numVertices;
		vbDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;
		vbDesc.CPUAccessFlags = 0;
		vbDesc.MiscFlags = 0;

		D3D11_SUBRESOURCE_DATA vbData = { 0 };
		vbData.pSysMem = data;

		return device->CreateBuffer(&vbDesc, &vbData, buffer.GetAddressOf());		
	}
};
```
