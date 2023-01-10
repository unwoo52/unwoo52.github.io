---
title: GetRawInputData
author: unwoo52
date: 2022-11-17 00:00:00 +09:00
categories: [Win32API]
tags: [Win32API, GetRawInputData]
---

## 코드

```cpp
UINT GetRawInputData(
  [in]            HRAWINPUT hRawInput,
  [in]            UINT      uiCommand,
  [out, optional] LPVOID    pData,
  [in, out]       PUINT     pcbSize,
  [in]            UINT      cbSizeHeader
);
```

## [in] hRawInput

Type: HRAWINPUT

RAWINPUT 구조 에 대한 핸들입니다 . 이것은 WM_INPUT 의 lParam 에서 가져옵니다 .

A handle to the RAWINPUT structure. This comes from the lParam in WM_INPUT.

## [in] uiCommand

Type: UINT

명령 플래그입니다. 이 매개변수는 다음 값 중 하나일 수 있습니다.

The command flag. This parameter can be one of the following values.

|Value|Meaning|
|------|---|
|RID_HEADER 0x10000005|Get the header information from the RAWINPUT structure.|
|RID_INPUT 0x10000003|Get the raw data from the RAWINPUT structure.|

|Value|Meaning|
|------|---|
|RID_HEADER 0x10000005|RAWINPUT 구조 에서 헤더 정보를 가져옵니다 .|
|RID_INPUT 0x10000003|RAWINPUT 구조 에서 원시 데이터를 가져옵니다 .|

## [out, optional] pData

Type: LPVOID

RAWINPUT 구조 에서 오는 데이터에 대한 포인터입니다 . 이것은 uiCommand 값에 따라 다릅니다 . pData 가 NULL 이면 필요한 버퍼 크기가 * pcbSize 에 반환됩니다 .

A pointer to the data that comes from the RAWINPUT structure. This depends on the value of uiCommand. If pData is NULL, the required size of the buffer is returned in *pcbSize.

## [in, out] pcbSize

Type: PUINT

pData 에 있는 데이터의 크기(바이트)입니다 .

The size, in bytes, of the data in pData.

## [in] cbSizeHeader

Type: UINT

RAWINPUTHEADER 구조 의 크기(바이트)입니다 .

The size, in bytes, of the RAWINPUTHEADER structure.




## 참고

[마이크로소프트 공식문서](https://learn.microsoft.com/ko-kr/windows/win32/api/winuser/nf-winuser-getrawinputdata)
