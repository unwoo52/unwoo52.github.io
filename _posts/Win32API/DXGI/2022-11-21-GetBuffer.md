---
title: GetBuffer
author: unwoo52
date: 2022-11-21 00:00:00 +09:00
categories: [Win32API]
tags: [Win32API]
---

## GetBuffer

```cpp
HRESULT GetBuffer(
        UINT   Buffer,
  [in]  REFIID riid,
  [out] void   **ppSurface
);
```

스왑 체인의 backBuffer중 하나에 액세스합니다.

## Parameters

>### Buffer
>
>Type: UINT
>
>A zero-based buffer index.
>
>If the swap chain's swap effect is DXGI_SWAP_EFFECT_DISCARD, this method can only access the first buffer; for this situation, set the index to zero.
>
>If the swap chain's swap effect is either DXGI_SWAP_EFFECT_SEQUENTIAL or DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL, only the swap chain's zero-index buffer can be read from >and written to. The swap chain's buffers with indexes greater than zero can only be read from; so if you call the IDXGIResource::GetUsage method for such buffers, >they have the DXGI_USAGE_READ_ONLY flag set.
>
>0부터 시작하는 버퍼 인덱스입니다.
>
>스왑 체인의 스왑 효과가 DXGI_SWAP_EFFECT_DISCARD 인 경우 이 메서드는 첫 번째 버퍼에만 액세스할 수 있습니다. 이 경우 인덱스를 0으로 설정하십시오.
>
>스왑 체인의 스왑 효과가 DXGI_SWAP_EFFECT_SEQUENTIAL 또는 DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL 이면 스왑 체인의 제로 인덱스 버퍼만 읽고 쓸 수 있습니다. 인덱스가 0보다 큰 스왑 체인>의 버퍼는 읽을 수만 있습니다. 따라서 이러한 버퍼에 대해 IDXGIResource::GetUsage 메서드를 호출하면 DXGI_USAGE_READ_ONLY 플래그가 설정됩니다.
>
>### [in] riid
>
>Type: REFIID
>
>The type of interface used to manipulate the buffer.
>
>버퍼를 조작하는 데 사용되는 인터페이스 유형입니다.
>
>### [out] ppSurface
>
>Type: void**
>
>A pointer to a back-buffer interface.
>
>백 버퍼 인터페이스에 대한 포인터입니다.



## Return value
>Type: HRESULT
>
>Returns one of the following [DXGI_ERROR](https://learn.microsoft.com/en-us/windows/win32/direct3ddxgi/dxgi-error).
>
>다음 [DXGI_ERROR](https://learn.microsoft.com/en-us/windows/win32/direct3ddxgi/dxgi-error) 중 하나를 반환합니다 .

## 참고

[마이크로소프트 공식문서](https://learn.microsoft.com/en-us/windows/win32/api/dxgi/nf-dxgi-idxgiswapchain-getbuffer)
