---
layout: post
title:  "(Win32 : WindowsProgramming-18) Thread Synchronization - 1"
summary: ""
author: win32
date: '2021-05-12 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-18/
---

## Critical Section

```cpp
#include <stdio.h>
#include <Windows.h>
#include <process.h>
#include <tchar.h>

void delay() { for (int i = 0; i < 2000000; i++); }

UINT __stdcall foo(void* p)
{
    int x = 0;

    for(int i = 0; i < 20; i++)
    {
        x = 100; delay();
        x = x+1; delay();
        printf("%s : %d\n", p, x);
    }

    return 0;
}

int main()
{
    HANDLE h1 = (HANDLE)_beginthreadex(0, 0, foo, (void*)"A", 0, 0);
    HANDLE h2 = (HANDLE)_beginthreadex(0, 0, foo, (void*)"\tB", 0, 0);

    HANDLE h[2] = { h1, h2 };
    WaitForMultipleObjects(2, h , TRUE, INFINITE);
    CloseHandle(h1);
    CloseHandle(h2);
    return 0;
}
```

* 지역변수는 스택에 놓이고
* 스택은 스레드당 한 개씩 따로 만들어진다.
* 결론적으로 x는 A, B Thread가 같이 쓰지 않는다.

---

```cpp
// ...

UINT __stdcall foo(void* p)
{
    static int x = 0;

    for(int i = 0; i < 20; i++)
    {
        x = 100; delay();
        x = x+1; delay();
        printf("%s : %d\n", p, x);
    }

// ...
```

* static 지역변수 또는 전역변수는 data메모리 공간에 놓인다.
* 모든 Thread가 데이터를 공유하게 된다.
* -> Critical Section의 필요성

해결책?

```cpp
#include <stdio.h>
#include <Windows.h>
#include <process.h>
#include <tchar.h>

void delay() { for (int i = 0; i < 2000000; i++); }

// 1. 전역 구조체 만들기
CRITICAL_SECTION cs;

UINT __stdcall foo(void* p)
{
    int x = 0;

    for(int i = 0; i < 20; i++)
    {
        EnterCriticalSection(&cs);  // CriticalSection Enter
        x = 100; delay();
        x = x+1; delay();
        printf("%s : %d\n", p, x);
        LeaveCriticalSection(&cs);  // CriticalSection Leave
    }

    return 0;
}

int main()
{
    InitializeCriticalSection(&cs); // CriticalSection 초기화

    HANDLE h1 = (HANDLE)_beginthreadex(0, 0, foo, (void*)"A", 0, 0);
    HANDLE h2 = (HANDLE)_beginthreadex(0, 0, foo, (void*)"\tB", 0, 0);

    HANDLE h[2] = { h1, h2 };
    WaitForMultipleObjects(2, h , TRUE, INFINITE);
    CloseHandle(h1);
    CloseHandle(h2);

    DeleteCriticalSection(&cs); // CriticalSection 삭제

    return 0;
}
```

이런 경우가 발생한다. CriticalSection에 들어가려다 현재 사용중이라 Block을 했는데 Block을 하는 도중 CriticalSection에 들어갈 수 있는 상황이 되었다.<br>
이럴 경우 리소스를 비효율적으로 사용하게 되는데 Block이 되면서 Block이 된 정보를 어딘가 담을것이고 다시 Block에서 Active로 돌리면서 또 정보를 담아야 하는데 이 모든것이 오버로드이다.<br>
**차라리 CriticalSection에 들어가려는 시도를 여러번해보자.**

```cpp
#include <stdio.h>
#include <Windows.h>
#include <process.h>
#include <tchar.h>

void delay() { for (int i = 0; i < 2000000; i++); }

// 1. 전역 구조체 만들기
CRITICAL_SECTION cs;

UINT __stdcall foo(void* p)
{
    int x = 0;

    for(int i = 0; i < 20; i++)
    {
        EnterCriticalSection(&cs);
        x = 100; delay();
        x = x+1; delay();
        printf("%s : %d\n", p, x);
        LeaveCriticalSection(&cs);
    }

    return 0;
}

int main()
{
    InitializeCriticalSectionAndSpinCount(&cs, 4000);   // 들어가려는 시도를 4000번 해봐라

    HANDLE h1 = (HANDLE)_beginthreadex(0, 0, foo, (void*)"A", 0, 0);
    HANDLE h2 = (HANDLE)_beginthreadex(0, 0, foo, (void*)"\tB", 0, 0);

    HANDLE h[2] = { h1, h2 };
    WaitForMultipleObjects(2, h , TRUE, INFINITE);
    CloseHandle(h1);
    CloseHandle(h2);

    DeleteCriticalSection(&cs);

    return 0;
}
```

---

## Mutex 개념

* 하나의 화장실에 여러명이 들어가려한다.
* 키는 하나 뿐이고 이 키를 Mutex라 하자
* 누군가 화장실을 사용중일때는 나머지 인원은 대기해야 한다.

* Mutex : 공유 자원을 하나의 Thread가 독점할 수 있게 해줌

```cpp
// MutexExample.exe
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

int main()
{
    printf("try acquire mutex\n");

    HANDLE hMutex = CreateMutexEx(0, _T("MyMutex"), 0, MUTEX_ALL_ACCESS);

    WaitForSingleObject(hMutex, INFINITE);  // 소유자가 없을 경우 Signal

    // wait로 mutex 통과시, Thread가 mutex를 소유하게 됨.

    printf("get mutex\n");

    MessageBoxA(0, "release", "", MB_OK);
    ReleaseMutex(hMutex);

    CloseHandle(hMutex);
}
```

`MutexExample.exe`를 여러번 실행해보면 첫 번째를 제외하고 이후는 get mutex를 볼 수 없다.<br>
먼저 Mutex를 잡았던 프로세스가 Mutex를 Release해줘야 한다.<br>

(참고) Windows는 같은 Kernel Object 이름을 지원하지 않음, 같은 Kernel Object로 만드려할 경우 기존에 있던 Kernel Object 를 리턴해준다.

---

```cpp
// MutexExample2.exe
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

int main()
{
    printf("try acquire mutex\n");

    HANDLE hMutex = CreateMutexEx(0, _T("MyMutex"), 0, MUTEX_ALL_ACCESS);

    WaitForSingleObject(hMutex, INFINITE);
    printf("get mutex\n");
    MessageBoxA(0, "release", "", MB_OK);

    WaitForSingleObject(hMutex, INFINITE);  // 주 Thread에서 Mutex를 갖고 있기에 여기서 Wait되지 않음
    printf("get mutex\n");
    MessageBoxA(0, "release", "", MB_OK);

    // 단, 하나의 Thread에서 Mutex를 두 번 소유할 경우
    // 소유 카운트가 올라가기에 Release도 두 번 해줘야 한다.
    ReleaseMutex(hMutex);
    ReleaseMutex(hMutex);

    CloseHandle(hMutex);
}
```

---

```cpp
// MutexExample3.exe
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

int main()
{
    printf("try acquire mutex\n");

    HANDLE hMutex = CreateMutexEx(0, _T("MyMutex"), 0, MUTEX_ALL_ACCESS);

    WaitForSingleObject(hMutex, INFINITE);

    printf("get mutex\n");

    MessageBoxA(0, "release", "", MB_OK);
    //ReleaseMutex(hMutex);
    // Mutex를 반납하지 않고 그냥 죽어버린다면??

    /*
    ABANDONED(버려진) MUTEX
    -> 뮤텍스를 소유한 스레드가 ReleaseMutex로 반납하지 않고 죽은경우
    -> 새로운 스레드가 Mutex를 소유해서 사용할 수 있지만 공유 자원에 문제가 있을 수 있다.
    */

    CloseHandle(hMutex);
}
```

```cpp
DWORD ret = WaitForSingleObject(hMutex, INFINITE);

if(ret == WAIT_OBJECT_0)
{
    // 정상종료되어 Mutex를 받음.
}
else if(ret == WAIT_ABANDONED)
{
    // 포기된 뮤텍스를 받음.
}
```

---

## Semaphore

* 화장실이 여러개이고, 그 화장실의 개수만큼 키를 만듦
* 키를 역시 Semaphore라 한다.
* 자원애 개수를 관리하고 자원의 한정적 공유가 가능해진다.

```cpp
// SemaphoreExample.exe
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

int main()
{
    HANDLE hSem = CreateSemaphoreEx(0,  // 보안 값
                    3,                  // 카운트 값
                    3,                  // 최대 카운트 값
                    _T("MySem"),        // 이름
                    0, 
                    SEMAPHORE_ALL_ACCESS);

    WaitForSingleObject(hSem, INFINITE);    // 카운트가 0보다 클 시 Signal
    // 카운트 값이 -1된다.
    // 만약 카운트 값이 0이 될시 non Signal이 됨.

    MessageBoxA(0, "Release", "", MB_OK);
    LONG old;
    ReleaseSemaphore(hSem, 1, &old);

    return 0;
}
```

`SemaphoreExample.exe`를 여러번 실행하면 카운트 만큼은 메시지박스가 나타나나<br>
카운트 이상넘어가면 non Signal 되어 메시지박스가 나타나지 않음.

---

## Event

* Thread간 통신에 사용된다.
* 하나의 Thread에서 작업이 완료되었음을 알릴 수 있다.

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>
#include <process.h>

HANDLE hEvent = 0;

UINT __stdcall foo(void* p)
{
    WaitForSingleObject(hEvent, INFINITE);  // Signal이 되지 않기에 무한 대기
    printf("foo start work\n");
    return 0;
}

int main()
{
    hEvent = CreateEventEx(0, _T("MyEvent"),
                0, // 초기 시그널 상태와 reset의 종류(0 : non signal, auto reset)
                EVENT_ALL_ACCESS);

    HANDLE hThread = (HANDLE)_beginthreadex(0, 0, foo, 0, 0, 0);

    getchar();
    SetEvent(hEvent);   // Signal 됨
    getchar();
    CloseHandle(hEvent);
    return 0;
}
```

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>
#include <process.h>

HANDLE hEvent = 0;

UINT __stdcall foo(void* p)
{
    // Wait를 두 번 하게 해보자.
    WaitForSingleObject(hEvent, INFINITE);
    printf("foo start work\n");

    // auto reset의 효과로 다시 non-Signal되어 여기서 멈춤
    WaitForSingleObject(hEvent, INFINITE);
    printf("foo start work\n");
    return 0;
}

int main()
{
    hEvent = CreateEventEx(0, _T("MyEvent"),
                0,
                EVENT_ALL_ACCESS);

    HANDLE hThread = (HANDLE)_beginthreadex(0, 0, foo, 0, 0, 0);

    getchar();
    SetEvent(hEvent);
    getchar();
    CloseHandle(hEvent);
    return 0;
}
```