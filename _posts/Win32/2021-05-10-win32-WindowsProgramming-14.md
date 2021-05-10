---
layout: post
title:  "(Win32 : WindowsProgramming-14) Kernel Object"
summary: ""
author: win32
date: '2021-05-10 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-14/
---

## Object Categories

A, B라는 별도의 프로세스가 존재할 시<br>

* A의 핸들을 B에서 받아서 사이즈 조정이 가능할까? -> 가능(윈도우 핸들은 public to all processes)
* A에서 만든 GUI 팬을 B에서 사용이 가능할까? -> 불가능(private to a process)
* A에서 만든 파일을 B에서 파일핸들을 통해 접근이 가능할까 -> 상황에 따라 다름(process specific)

세 가지 경우의 수가 있는데 어떤경우 인지 자세히 살펴보자.

---

* User Object
    * 윈도우와 관련된 Object
    * public to all processes
    * 파괴 함수 : DestroyXXX()
    * 관련 DLL : User32.dll
* GDI Object
    * 그래픽 관련 Object
    * private to a process
    * 파괴 함수 : DeleteXXX()
    * 관련 DLL : Gdi32.dll
* Kernel Object
    * 파일, 메모리, 프로세스등 UI 이외의 작업에 관련된 Object
    * process specific
    * 파괴 함수 : CloseHandleXXX()
    * 관련 DLL : Kernel.dll

* [참고사이트](https://docs.microsoft.com/en-us/windows/win32/sysinfo/object-categories)

---

## Kernel Object

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

int _tmain()
{
    HANDLE hFile = CreateFile(_T("a.txt"), /* ... */);
    _tprintf(_T("FILE HANDLE : %x\n"), hFile);

    // 다른 프로세스에 파일 핸들 전달
    HWND hwnd = FindWindow(0, _T("Friend"));
    SendMessage(hwnd, WM_APP+100, 0, (LPARAM)hFile);
}
```

일단 다른 프로세스에서 파일의 핸들을 사용하지 못한다.<Br>
넘겨준 핸들은 실제 물리적 주소값을 의미하는 것이 아니라 Kernel내부적으로 관리하는 Object Table내의 index이다.<br>
해당 index에서 파일이 접근 가능한지 쓸 수 있는지 등을 OS에서 알려주고 파일에 접근하게 된다.<br>
따라서 파일의 핸들(단순 index)만으론 파일에 접근이 불가능 하다

그래도 하고싶다면? -> `DuplicateHandle()`과 같은 Win32에서 제공하는 API를 사용해야 한다.
