---
layout: post
title:  "(Win32 : WindowsProgramming-32) Driver Example"
summary: ""
author: win32
date: '2021-05-17 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-32/
---

## Driver와 API

* Driver 소스에서는 Win32 API를 사용할 수 없다.
* Kernel API를 사용해야 한다.

* `printf()` : 내부적으로 Win32를 사용 사용불가
* `strlen()` : Win32를 사용하지 않지만 Driver에서는 ANSI가 아닌 UNICODE를 사용하기에 불편

---

## Driver Example

* 프로세스의 생성(`CreateProcess`), 종료(`ExitProcess/TerminateProcess`)를 감시하기 위한 API를 만들어보자.
* 참고로 Kernel API 중 (`PsSetCreateProcessNotifyRoutine`)을 이용시 프로세스의 생성/종료를 통보 받을 수 있다.

```cpp
#pragma warning(disable:4100)

#include <ntddk.h>

void foo(HANDLE ParentId, HANDLE ProcessId, BOOLEAN Create)
{
    if(Create)
        DbgPrint("Create Process : %d", ProcessId);
    else
        DbgPrint("Terminated Process : %d", ProcessId);
}

void DriverUnload(PDRIVER_OBJECT pDriver)
{
    DbgPrint("Ex2 : DriverUnload");
    PsSetCreateProcessNotifyRoutine(foo, TRUE);
}

extern "C" NTSTATUS DriverEntry(PDRIVER_OBJECT pDriver, PUNICODE_STRING pRegPath)
{
    DbgPrint("Ex2 : DriverEntry");

    pDriver->DriverUnload = DriverUnload;

    PsSetCreateProcessNotifyRoutine(foo, FALSE);

    return STATUS_SUCCESS;
}
```
