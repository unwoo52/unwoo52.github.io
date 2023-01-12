---
title: AdapterReader
author: unwoo52
date: 2022-11-18 00:00:00 +09:00
categories: [DirectX, GameEngine ]
tags: [directX, Win32API, GameEngine, AdapterReader]
---

## DXGI

그래픽카드와 DirectX간의 버전 차이로 인한 호환이 안될 수 있는 문제를 해결하기 위해 있는 둘 사이의 인터페이스. 인터페이스를 사용하기 때문에 결합이 느슨해지는 장점도 있다.

## AdapterReader.h

<details>
<summary markdown="span"> 
  
</summary>
  
AdapterReader.h 전체 코드
  
```cpp
#pragma once
#include "../DebugLog.h"
#include <d3d11.h>
#pragma comment(lib, "d3d11.lib")
#pragma comment(lib, "DirectXTK.lib")
#pragma comment(lib, "DXGI.lib")
#include <wrl/client.h>
#include <vector>
using namespace std;
using namespace Microsoft::WRL;
class AdapterData
{
public:
	AdapterData(IDXGIAdapter* pAdapter);
	IDXGIAdapter* pAdapter = nullptr;
	DXGI_ADAPTER_DESC description;
};
class AdapterReader
{
	static vector<AdapterData> adapters;
public:
	static vector<AdapterData> GetAdapters();
};
```
  
</details>

-------------

#### 코드 설명

```cpp
#include <d3d11.h>
```
DXGI include.

```cpp
#include <wrl/client.h>
```

c++에서 스마트 포인터는 memory를 include해서 사용했지만, DX에서는 COM프레임워크를 사용한다. 때문에 COM프레임워크의 스마트포인터를 이용하기 위해 wrl/client.h를 include한것이다.

```cpp
#pragma comment(lib, "d3d11.lib")
#pragma comment(lib, "DirectXTK.lib")
#pragma comment(lib, "DXGI.lib")
```

라이브러리 인클루드. 

pragma comment와 include의 차이점은 include는 h파일(코드 파일)을 가져오고, pragma comment는 라이브러(컴파일된 오브젝트 파일)를 가져온다.


## AdapterReader.cpp

<details>
<summary markdown="span"> 
AdapterReader.cpp 전체 코드
</summary>
#include "AdapterReader.h"

vector<AdapterData> AdapterReader::adapters;

AdapterData::AdapterData(IDXGIAdapter* pAdapter)
{
	this->pAdapter = pAdapter;
	HRESULT hr = pAdapter->GetDesc(&description);
	if (FAILED(hr))
	{
		Debug::Log(hr, L"어탭터 정보를 읽어 오는데 실패 하였습니다.");
	}
}

vector<AdapterData> AdapterReader::GetAdapters()
{
	if (adapters.size() > 0)
	{
		return adapters;
	}

	ComPtr<IDXGIFactory> pFactory;
	HRESULT hr = CreateDXGIFactory(__uuidof(IDXGIFactory), reinterpret_cast<void**>(pFactory.GetAddressOf()));
	if (FAILED(hr))
	{
		Debug::Log(hr, L"Failed to create DXGIFactory for enumerating adapters.");
		exit(-1);
	}

	IDXGIAdapter* pAdapter;
	UINT index = 0;
	while (SUCCEEDED(pFactory->EnumAdapters(index++, &pAdapter)))
	{
		adapters.push_back(AdapterData(pAdapter));		
	}

	return adapters;
}

</details>

----

#### 코드 설명

```cpp
vector<AdapterData> AdapterReader::adapters;
```

static 멤버를 초기화.

cs는 선언과 동시에 초기화가 가능하지만, cpp은 별도로 외부에서 정의를 통해 초기화를 해야 한다.

```cpp
AdapterData::AdapterData(IDXGIAdapter* pAdapter)
{
	this->pAdapter = pAdapter;
	HRESULT hr = pAdapter->GetDesc(&description);
	if (FAILED(hr))
	{
		Debug::Log(hr, L"어탭터 정보를 읽어 오는데 실패 하였습니다.");
	}
}
```

어뎁터 포인터를 인자로 받아 어뎁터를 이 클래스 멤버에 저장하는 메소드.

```cpp
vector<AdapterData> AdapterReader::GetAdapters()
{
	if (adapters.size() > 0)
		return adapters;
	ComPtr<IDXGIFactory> pFactory; //COM프레임워크에서 사용되는 포인터
	HRESULT hr = CreateDXGIFactory(__uuidof(IDXGIFactory), reinterpret_cast<void**>(pFactory.GetAddressOf()));
	if (FAILED(hr)) 
    	...Fail code
	IDXGIAdapter* pAdapter;
	UINT index = 0;
	while (SUCCEEDED(pFactory->EnumAdapters(index++, &pAdapter)))
	{
		adapters.push_back(AdapterData(pAdapter));		
	}
	return adapters;
}
```

컴퓨터 내의 모든 그래픽카드들(adapters)을 찾아서 반환하는 함수. 그래픽카드가 여러개면 adapters벡터에 저장되며 인덱스가 갯수만큼 증가한다.



