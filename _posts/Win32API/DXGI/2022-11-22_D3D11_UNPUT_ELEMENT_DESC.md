---
title: D3D11_INPUT_ELEMENT_DESC
author: unwoo52
date: 2022-11-22 00:00:00 +09:00
categories: [C++]
tags: [C++, directX, Win32API, GameEngine ]
---


```cpp
{"POSITION", 0, DXGI_FORMAT::DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D11_INPUT_CLASSIFICATION::D3D11_INPUT_PER_VERTEX_DATA, 0}
```

### LPCSTR : 시멘틱 이름. string값으로 전달하여 어떤 시멘틱에 대한 정보를 정의할지를 알려준다.

### UINT : 시멘틱에 대한 인덱스

### DXGI_FORMAT : [DXGI_FORMAT](https://learn.microsoft.com/en-us/windows/win32/api/dxgiformat/ne-dxgiformat-dxgi_format)

### 유형: UINT

입력 어셈블러를 식별하는 정수 값입니다(입력 슬롯 참조). 유효한 값은 D3D11.h에 정의된 0에서 15 사이입니다.

AlignedByteOffset

### 유형: UINT

선택 과목. 꼭짓점의 시작부터 오프셋(바이트)입니다. 편의를 위해 D3D11_APPEND_ALIGNED_ELEMENT를 사용하여 필요한 경우 패킹을 포함하여 이전 요소 바로 뒤에 현재 요소를 정의합니다.

InputSlotClass

### 유형: D3D11_INPUT_CLASSIFICATION

단일 입력 슬롯에 대한 입력 데이터 클래스를 식별합니다( D3D11_INPUT_CLASSIFICATION 참조 ).

InstanceDataStepRate

### 유형: UINT

버퍼에서 한 요소씩 진행하기 전에 동일한 인스턴스별 데이터를 사용하여 그릴 인스턴스 수입니다. 정점별 데이터를 포함하는 요소의 경우 이 값은 0이어야 합니다(슬롯 클래스는 D3D11_INPUT_PER_VERTEX_DATA로 설정됨).


## 사용한 예시

```cpp
typedef struct D3D11_INPUT_ELEMENT_DESC {
  LPCSTR                     SemanticName;
  UINT                       SemanticIndex;
  DXGI_FORMAT                Format;
  UINT                       InputSlot;
  UINT                       AlignedByteOffset;
  D3D11_INPUT_CLASSIFICATION InputSlotClass;
  UINT                       InstanceDataStepRate;
} D3D11_INPUT_ELEMENT_DESC;
```


