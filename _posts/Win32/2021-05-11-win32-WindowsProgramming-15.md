---
layout: post
title:  "(Win32 : WindowsProgramming-15) Process ID, Handle"
summary: ""
author: win32
date: '2021-05-11 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-15/
---

## Process ID 구하는 방법

```cpp
// 자신의 Process ID
DWORD pid = GetCurrentProcessId();
DWORD tid = GetCurrentThreadId();
```

```cpp
// 다른 Process의 Process ID
DWORD pid = 0;
DWORD tid = 0;

// 핸들을 알아야 받을 수 있다.
HWND hwnd = FindWindow(0, _T("계산기"));

tid = GetWindowThreadProcessId(hwnd, &pid);
```

---

## Process ID, Process Handle 차이점?

* 한 프로그램의 프로세스ID는 모든 프로그램에서 동일하다
* 단 해당 프로그램의 프로세스 ID를 안다고 어떠한 컨트롤을 할 수 있는 것은 아니다
* 프로세스 ID를 기반으로 OS에게 핸들을 받아서 어떠한 처리가 가능하다
* OS에게 받은 핸들값은 핸들을 요청한 프로그램 마다 다르게 할당해준다.

```cpp
DWORD pid = 0;
DWORD tid = 0;

HWND hwnd = FindWindow(0, _T("계산기"));

tid = GetWindowThreadProcessId(hwnd, &pid);

// 프로세스 핸들 얻기
HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, 0, pid);

getchar();

// 프로세스 죽여보기
TerminateProcess(hProcess, 0);

// Close를 무조건 해야하는 걸로 봐서
// Process Kernel Object Table에 핸들이 추가 된다.
CloseHandle(hProcess);
```

---

## Example - 다른 프로세스의 메모리 읽기

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

int _tmain()
{
    DWORD pid = GetCurrentProcessId();  // 이 process id는 다른 프로세스에서 사용됨
    char passwd[256] = { 0 };           // 이 메모리 주소는 다른 프로세스에서 사용됨

    while(1)
    {
        gets_s(passwd);
        // 콘솔에 문자입력 후 엔터를 치면 passwd에 들어가게 된다.
    }
}
```

다른 프로세스에서 `passwd`에 기록된 데이터를 읽어보자.

```cpp
int _tmain()
{
    char buff[256] = { 0 };

    DWORD pid = 22060;              // pid, 하드코딩
    char* addr = (char*)0x010FFA74; // 메모리 주소, 하드코딩

    HANDLE hProcess = OpenProcess(PROCESS_VM_READ, 0, pid);

    while(1)
    {
        getchar();
        DWORD len;
        ReadProcessMemory(hProcess, addr, buff, 256, &len);
        printf("읽어온 메모리 : %s\n", buff);
    }
}
```

프로세스 ID는 API로 구할수 있으나<br>
메모리 주소의 경우 어떻게 구하나? -> 디버깅 툴을 이용해서 구해야 한다.(나중에 설명예정)