---
layout: post
title:  "(Win32 : WindowsProgramming-31) Hello, Driver"
summary: ""
author: win32
date: '2021-05-17 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-31/
---

## 개발 도구

* Visual Stduio
* Windows Driver Kit
* Windows Driver Kit "Visual Studio Extension"

![](/assets/img/posts/win32/WindowsProgramming-31-1.PNG){:class="img-fluid"}

* Device, Driver의 경우 BlueScreen을 유발할 수 있기에 VM을 통해서 테스트해보도록 하자.

---

## 프로젝트생성

![](/assets/img/posts/win32/WindowsProgramming-31-2.PNG){:class="img-fluid"}

* 자동 생성된 .inf 파일 제거
* 자신의 OS에 맞게 x84, x64 선택
* 프로젝트 메뉴 -> 속성 -> C/C++ -> 코드 생성 항목 선택 후 -> 스펙터 완화(Spectre Mitigration) 라이브러리 사용 안함 선택

```cpp
// ex1.cpp
#pragma warning(disable:4100)

#include <ntddk.h>

void DriverUnload(PDRIVER_OBJECT pDriver)
{
    DbgPrint("DriverUnload");
}

extern "C" NTSTATUS DriverEntry(PDRIVER_OBJECT pDriver, PUNICODE_STRING pRegPath)
{
    DbgPrint("DriverEntry : &p\n", pDriver);

    pDriver->DriverUnload = DriverUnload;

    return STATUS_SUCCESS;
}
```

빌드하면 `ex1.sys`파일이 생성된다.

---

## Driver Install

이제 아래의 단계들이 남았다

* 디바이스 드라이버를 시스템에 설치
* 디바이스 드라이버 시동(Start)
* 디바이스 드라이버 중지(Stop)
* 디바이스 드라이버 시스템에서 제거

### 디바이스 드라이버를 시스템에 설치

레지스트리 `컴퓨터\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services`에 접근 여기에 드라이버를 설치한다. 예를 들어서

![](/assets/img/posts/win32/WindowsProgramming-31-3.PNG){:class="img-fluid"}

`컴퓨터\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\ACPI`에 접근시 ACPI.sys가 어디에 설치되어있는지 알 수 있다.

이제 레지스트리를 추가해야 하는데 방법은 세 가지가 있다.

* 직접수정(비추천, 특히 Device Driver를 추가하기에 위험)
* 레지스트리 API이용
* sc.exe(SystemControll.exe) 도구 사용(이 방법을 이용할 예정)

* 관리자 권한 cmd실행
* `sc create Ex1 type=kernel binPath=C:\Driver\Ex1\x64\debug\Ex1.sys`

여기까지하면 레지스트리에 등록을 확인 가능

### 디바이스 드라이버 시동(Start)

드라이버를 메모리에 올리는 과정이다.

```
sc start Ex1
```

단, 디지털서명이 없으면 드라이버 시동이 불가능 윈도우 부트 설정을 변경해야 한다.<br>
관리자 모드 cmd에서

```
bcdedit /set testsigning on
```

### 디바이스 드라이버 중지(Stop)

```
sc stop Ex1
```

### 디바이스 드라이버 시스템에서 제거

```
sc delete Ex1
```

---

## Device Driver 코드

```cpp
// ex1.cpp
#pragma warning(disable:4100)

#include <ntddk.h>  // 디바이스 드라이버를 개발하기 위한 함수/구조체 포함

// void DriverUnload([[maybe_unused]] PDRIVER_OBJECT pDriver) // (C++17) warning 4100을 방지해 준다.
void DriverUnload(PDRIVER_OBJECT pDriver)
{
    UNREFERENCED_PARAMETER(pDriver);        // warning 4100을 방지해 준다.

    DbgPrint("DriverUnload");
    // DbgView.exe에서 출력을 확인가능

    KdPrint(("AAAA"));
    // DBG가 정의되어 있다면 DbgPrint로 변환해줌
}

// Driver Entry Point (main이라 생각)
extern "C" NTSTATUS DriverEntry(PDRIVER_OBJECT pDriver,     // OS에서 관리하는 DeviceDriver 객체 주소
                                // DeviceDriver를 생성시 마다 할당된다.
                                PUNICODE_STRING pRegPath)
{
    DbgPrint("DriverEntry : &p\n", pDriver);

    // Unload될때 어떤 함수를 부를지 등록(반드시 해야한다.)
    pDriver->DriverUnload = DriverUnload;

    return STATUS_SUCCESS;
}
```