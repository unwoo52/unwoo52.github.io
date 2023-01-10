---
title: Vertex Shader and Pixel Shader
author: unwoo52
date: 2022-11-21 00:00:00 +09:00
categories: [DirectX, GameEngine ]
tags: [directX, Win32API, DXGI, GameEngine, Graphics, Shaders, VertexShader, PixelShader]
---

## Shaders

### Shader.h

```cpp
class VertexShader
{
	ComPtr<ID3D11VertexShader> shader;
	ComPtr<ID3D10Blob> buffer;
	ComPtr<ID3D11InputLayout> inputLayout;
public:
	bool Initialize(ID3D11Device* device, wstring path, D3D11_INPUT_ELEMENT_DESC* layout, UINT numElements);
	ID3D11VertexShader* GetShader();
	ID3D10Blob* GetBuffer();
	ID3D11InputLayout* GetInputLayout();
};

class PixelShader
{
	ComPtr<ID3D11PixelShader> shader;
	ComPtr<ID3D10Blob> buffer;
public:
	bool Initialize(ID3D11Device* device, wstring path);
	ID3D11PixelShader* GetShader();
	ID3D10Blob* GetBuffer();
};
```

### Shader.cpp

```cpp
bool VertexShader::Initialize(ID3D11Device* device, wstring path, D3D11_INPUT_ELEMENT_DESC* layout, UINT numElements)
{
	HRESULT hr = D3DReadFileToBlob(path.c_str(), buffer.GetAddressOf());
	if (FAILED(hr)) { ... }

	hr = device->CreateVertexShader(buffer->GetBufferPointer(), buffer->GetBufferSize(), NULL, shader.GetAddressOf());
	if (FAILED(hr)) { ... }

	hr = device->CreateInputLayout(layout, numElements, buffer->GetBufferPointer(), buffer->GetBufferSize(), inputLayout.GetAddressOf());
	if (FAILED(hr)) { ... }

	return true;
}

bool PixelShader::Initialize(ID3D11Device* device, wstring path)
{
	HRESULT hr = D3DReadFileToBlob(path.c_str(), buffer.GetAddressOf());
	if (FAILED(hr)) { ... }

	hr = device->CreatePixelShader(buffer->GetBufferPointer(), buffer->GetBufferSize(), NULL, shader.GetAddressOf());
	if (FAILED(hr)) { ... }

	return true;
}
```



## hlsl

### hlsl 개요

```hlsl
float4 main(float2 inPos : POSITION) : SV_POSITION
{
	return float4(inPos, 0, 1);
}
```

- **렌더링 파이프 라인을 프로그래밍하고, 렌더링효과 계산에 쓴다. HLSL(High Level Shading Language), 고수준 셰이딩 언어, 셰이딩 할때 쓴다(OpenGL은 GLSL). 문법이 C++과 비슷하여 프로그래머라면 쉽게 배우고 쓸수 있다.**

코드에서 보이는 **\:** 뒤에 있는 값은 시멘틱이라고 부른다(식별자 같은 것).

(float2 inPos : POSITION)에서 float2 inPos은 POSITION에 해당하는 값이다 라고 말하는 것.

SV_POSITION의 SV는 ScreenView, 즉 스크린뷰 상의 포지션이다. float4가 이 SV_POSITION으로 반환된다.

IA VS RS PS OM 으로 이루어지는 렌더링 파이프 라인 단계중, RS단계를 기점으로 픽셀화를 수행하기 시작한다. 즉 RS전단계에서 3D로 렌더링 된 것을 이제 화면에 출력하기 위해 2D로 바꾸는 것!

### Vertex Shader Code

- vs.hlsl

```hlsl
cbuffer transform : register(b0)
{
    float4x4 mat;
}
struct VS_INPUT
{
    float3 pos : POSITION;
    float3 color : COLOR;
    float2 uv : TEXCOORD;
};
struct PS_INPUT
{
    float4 pos : SV_POSITION;
    float3 color : COLOR;
    float2 uv : TEXCOORD;
};
PS_INPUT vs(VS_INPUT inPut)
{
    PS_INPUT outPut;
        
    outPut.pos = mul(float4(inPut.pos, 1.0f), mat);
    outPut.color = inPut.color;
    outPut.uv = inPut.uv;
	return outPut;
}
```

### Pixel Shader Code

- ps.hlsl

```hlsl
cbuffer Alpha : register(b0)
{
    float alpha;
}
struct PS_INPUT
{
    float4 pos : SV_POSITION;
    float3 color : COLOR;
    float2 uv : TEXCOORD;
};
Texture2D tex : TEXTURE : register(t0);
SamplerState texSample : SAMPLER : register(s0);
float4 ps(PS_INPUT inPut) : SV_TARGET
{
    float3 color = tex.Sample(texSample, inPut.uv);
    color = color * inPut.color;
	return float4(color, alpha);    
}
```

ps(main함수)의 반환값이 float4이다. color값으로 (r g b a)의 모습이기 때문.

## InitializeShaders

- bool Graphics::InitializeShaders()

```cpp
D3D11_INPUT_ELEMENT_DESC layout[] =
    {
        {"POSITION", 0, DXGI_FORMAT::DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D11_INPUT_CLASSIFICATION::D3D11_INPUT_PER_VERTEX_DATA, 0},
        //DXGI_FORMAT_R32G32B32_FLOAT : R32G32B32 float의 크기를 반환. R32G32B32는 위에 있는 position의 크기.
        {"COLOR", 0, DXGI_FORMAT::DXGI_FORMAT_R32G32B32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_CLASSIFICATION::D3D11_INPUT_PER_VERTEX_DATA, 0},
    };

    UINT numElements = ARRAYSIZE(layout);    

    if (!vs.Initialize(device.Get(), shaderfolder + L"vs.cso", layout, numElements))
    {
        return false;
    }

    if (!ps.Initialize(device.Get(), shaderfolder + L"ps.cso"))
    {
        return false;
    }
```

Shaders vs와 ps의 InitializeShaders()에서 쉐이더를 사용하기 전에 layout과 size를 지정하는 코드이다.

COLOR의 배열(5번째 줄)의 5번째 인자가 POSITION의 크기이다. 이는 layout 데이터 덩어리에서 몇바이트까지가 POSITION값의 정보이고 몇바이터부터 COLOR값의 정보인지를 알려주는 역할을 한다.




## hlsl 사용하기

### hlsl의 IntelliSense 활성화 하기

IntelliSense는 코드를 읽기 좋게 분류에 따라 텍스트의 색을 각기 다르게 활성화 하는 기능이다. 확장을 통해 hlsl언어의 IntelliSense기능을 활성화 해보자.

![imagename](/assets/image/DirectX/GameEngine/Virtual-Shader/001.png)

확장을 설치한 후 비주얼스튜디오를 재실행하면 적용이 되어있다.

### 진입점 이름 변경하기

진입점 이름은 default로 main이 되어있지만, 다른 이름을 사용할 수 있다.

![imagename](/assets/image/DirectX/GameEngine/Virtual-Shader/002.png)

### 쉐이더 컴파일 하기

![imagename](/assets/image/DirectX/GameEngine/Virtual-Shader/003.png)

만든 쉐이더는 바로 사용할 수 없고 컴파일을 통해 사용해야만 한다.


![imagename](/assets/image/DirectX/GameEngine/Virtual-Shader/004.png)

그러면 위와 같은 cso 파일이 생성된다.

```cpp
vs.Initialize(device.Get(), shaderfolder + L"vs.cso", layout, numElements
```

이제 Graphics에서 쉐이더를 호출해 사용할 수 있게 되었다.



