---
layout: post
title:  "(Win32 : WindowsProgramming-28) DLL_INJECTION"
summary: ""
author: win32
date: '2021-05-16 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-28/
---

내가 만든 DLL을 실행중인 다른 프로세스에 넣어보자

* DLL Injection 방법
    * Remote Thread (우리가 사용할 방법)
    * Windows Message Hook
    * Debugging API Injection
    * Registry Injection
    * Trojan horse 

---

## Remote Thread

* `CreateRemoteThread`를 이용하면 다른 프로세스의 스레드를 생성할 수 있다
* 다른 프로세스에 생성된 스레드를 이용해 `LoadLibrary`를 호출해보자.

```cpp
// Spy.cpp
#include <Windows.h>
#include <stdio.h>

BOOL _stdcall DllMain(HINSTANCE hinstDll, DWORD fwdReason, LPVOID lpvReserved)
{
    switch(fwdReason)
    {
    case DLL_PROCESS_ATTACH:
        MessageBoxA(0, "Inject Spy.dll", "SPY" MB_OK);
        break;
    case DLL_PROCESS_DETACH:
        MessageBoxA(0, "Eject Spy.dll", "SPY" MB_OK);
        break;
    }

    return TRUE;
}
```

```cpp
#include <Windows.h>
#include <stdio.h>

int main()
{
    const char* dllname = "D:\\Spy.dll";

    // 1. Inject시킬 프로세스의 핸들 얻기
    HWND hwnd = FindWindowA("Notepad", 0);
    DWORD pid;l
    DWORD tid = GetWindowThreadProcessId(hwnd, &pid);
    HANDLE hProcess = OpenProcess(PRICESS_ALL_ACCESS, FALSE, pid);

    printf("%x, %x, %x\n" hwnd, pid, hProcess);

    // 2. LoadLibrary 주소 구하기
    // kernel32.dll내에 LoadLibrary가 있을 것이다.
    HMODULE hDll = GetModuleHandleA("kernel32.dll");
    PTHREAD_START_ROUTINE f = (PTHREAD_START_ROUTINE)GetProcAddress(hDll, "LoadLibraryA");

    // 여기서 구한 LoadLibrary주소가 내가 사용중인 Process의 LoadLibrary주소이지
    // Injection 시킬 Process의 LoadLibrary일 것이라는 보장이 없음
    // 이 부분에 대해선 이후에 설명, 우선은 Kernel32.dll 내에서 같은 주소를 쓸꺼라 가정하자

    printf("%p, %p\n", hDll, f);

    // 3. Inject 시킬 프로세스 내에서 스래드 생성 후 LoadLibraryA 실행하기
    // HANDLE hThread = CreateRemoteThread(hProcess, 0, 0, f, (void*)dllname, 0, 0);
    // (void*)dllname -> dll이름을 주소값으로 넘기는데 Inject 시킬 프로세스는 이 주소공간에 메모리가 할당되어 있지 않음 -> 메모리 복사 필요

    char* p = (char*)VirtualAllocEx(hProcess, 0, strlen(dllname)+1, MEM_RESERVE | MEM_COMMIT, PAGE_READWRITE);
    SIZE_T sz;
    WriteProcessMemory(hProcess, p, dllname ,strl(dllname)+1, &sz);

    HANDLE hThread = CreateRemoteThread(hProcess, 0, 0, f, (void*)p, 0, 0);

    // 4. 빌드
    // cl DLL_inject.cpp /link user32.lib kernel32.lib
}
```