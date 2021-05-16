---
layout: post
title:  "(Win32 : WindowsProgramming-29) Async IO"
summary: ""
author: win32
date: '2021-05-16 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-29/
---

```cpp
#include <Windows.h>
#include <stdio.h>
#include <conio.h>

int main()
{
    // 계산기 파일을 오픈
    HANDLE hFile = CreateFileA("C:\\Windows\\system32\\calc.exe",
                                GENERIC_READ, FILE_SHARE_READ,
                                0, OPEN_EXISTING,
                                FILE_ATTRIBUTE_NORMAL, 0);

    if(hFile == INVALID_HANDLE_VALUE)
    {
        printf("Error in Open File\n");
    }

    DWORD len = 0;
    char buff[4096] = {0};
    // 버퍼에 계산기 파일을 읽은 데이터를 쓴다
    BOOL ret = ReadFile(hFile, buff, 4096, &len, 0);

    // ReadFile를 통해 버퍼에 쓰는 시간이 걸릴텐데 버퍼에 다 쓰여진 후 다음 라인으로 넘어가게 된다.
    // 이런 방식을 동기(Synchronous)화 방식 이라한다.

    if(ret == FALSE)
    {
        printf("Read Error\n");
    }
    else
    {
        printf("Comleting writes synchronosly\n");
    }

    CloseHandle(hFile);
    _getch();
}
```

여기서 단점은 I/O작업(`ReadFile/WriteFile`)은 CPU와는 상관없이 I/O 제어칩에 의해서 처리되는데 CPU자원의 낭비라할 수 있다.

비동기 방식으로 처리하면 성능의 향상을 가져올 수 있다.

```cpp
#include <Windows.h>
#include <stdio.h>
#include <conio.h>

int main()
{
    HANDLE hFile = CreateFileA("C:\\Windows\\system32\\calc.exe",
                                GENERIC_READ, FILE_SHARE_READ,
                                0, OPEN_EXISTING,
                                FILE_FLAG_OVERLAPPED    // 비동기를 위한 옵션
                                , 0);

    if(hFile == INVALID_HANDLE_VALUE)
    {
        printf("Error in Open File\n");
    }

    char buff[4096] = {0};

    // 비동기를 위해서 OVERLAPPED 구조체 생성
    OVERLAPPED ov = { 0 };

    // 비동기로 할시 ret은 무조건 FALSE
    BOOL ret = ReadFile(hFile, buff, 4096, 0, &ov);


    if(ret == FALSE)
    {
        if(GetLastError() == ERROR_IO_PENNDING)
            printf("Read Error\n");
    }
    else
    {
        printf("Comleting writes synchronosly\n");
    }

    CloseHandle(hFile);
    _getch();
}
```

ReadFile이 끝난경우의 처리?

네 가지 방법이 존재한다.

* Device Signal 대기
* Event Kernel Object Signal 대기
* IO Callback 함수 사용
* IOCP (다음강에 별도로 설명)

### Device Kernel Object Signal 대기 방법

File Kernel Object는 I/O 작업이 시작하기 전에 non signal<br>
I/O 작업이 완료되면 Signal 상태로 변경된다.

이 방식의 단점은 하나의 파일에 여러개의 비동기 I/O를 요청한 경우 구별이 불가능하다

```cpp
#include <Windows.h>
#include <stdio.h>
#include <conio.h>

int main()
{
    HANDLE hFile = CreateFileA("C:\\Windows\\system32\\calc.exe",
                                GENERIC_READ, FILE_SHARE_READ,
                                0, OPEN_EXISTING,
                                FILE_FLAG_OVERLAPPED, 
                                0);

    if(hFile == INVALID_HANDLE_VALUE)
    {
        printf("Error in Open File\n");
    }

    char buff[4096] = {0};
    OVERLAPPED ov = { 0 };

    BOOL ret = ReadFile(hFile, buff, 4096, 0, &ov);


    if(ret == FALSE)
    {
        if(GetLastError() == ERROR_IO_PENNDING)
            printf("Read Error\n");

        // File Kernel Object가 Signal될때까지 대기
        WiatForSingleObject(hFile, INFINITE);
        printf("complet io\n");
    }
    else
    {
        printf("Comleting writes synchronosly\n");
    }

    CloseHandle(hFile);
    _getch();
}
```

### Event Kernel Object Signal 대기

OVERLAPPED 구조체에 event 등록

여러개의 File I/O를 처리가능

```cpp
#include <Windows.h>
#include <stdio.h>
#include <conio.h>

int main()
{
    HANDLE hFile = CreateFileA("C:\\Windows\\system32\\calc.exe",
                                GENERIC_READ, FILE_SHARE_READ,
                                0, OPEN_EXISTING,
                                FILE_FLAG_OVERLAPPED, 
                                0);

    if(hFile == INVALID_HANDLE_VALUE)
    {
        printf("Error in Open File\n");
    }

    char buff[4096] = {0};
    OVERLAPPED ov = { 0 };
    // Event 등록
    ov.hEvent = CreateEvent(0, 0, 0, 0);

    BOOL ret = ReadFile(hFile, buff, 4096, 0, &ov);

    OVERLAPPED ov2 = { 0 };
    ov2.Offset = 4096;              // Offset도 넣을 수 있다.
    ov2.hEvent = CreateEvent(0, 0, 0, 0);

    BOOL ret2 = ReadFile(hFile, buff, 4096, 0, &ov);


    if(ret == FALSE)
    {
        if(GetLastError() == ERROR_IO_PENNDING)
            printf("Read Error\n");

        // Event를 대기
        WaitForSingleObject(ov.hEvent, INFINITE);
    }
    else
    {
        printf("Comleting writes synchronosly\n");
    }

    CloseHandle(hFile);
    _getch();
}
```

### IO Callback 함수 사용

이 방식도 잘 사용하지 않는게, 비동기를 요청한 스레드가 CallBack함수를 실행하게 된다.

다른 방식의 경우 비동기를 요청한 스레드가 아닌 다른 스레드가 대기할 수 있다.

```cpp
#include <Windows.h>
#include <stdio.h>
#include <conio.h>

void __stdcall foo(DWORD error, DWORD bytes, LPOVERLAPPED ov)
{
    printf("foo\n");
    printf("error : %d\n", error);
    printf("bytes : %d\n", bytes);
}

int main()
{
    HANDLE hFile = CreateFileA("C:\\Windows\\system32\\calc.exe",
                                GENERIC_READ, FILE_SHARE_READ,
                                0, OPEN_EXISTING,
                                FILE_FLAG_OVERLAPPED, 
                                0);

    if(hFile == INVALID_HANDLE_VALUE)
    {
        printf("Error in Open File\n");
    }

    char buff[4096] = {0};
    OVERLAPPED ov = { 0 };

    // ReadFileEx를 사용, ReadFileEx는 비동기 전용
    // BOOL ret = ReadFile(hFile, buff, 4096, 0, &ov);
    BOOL ret = ReadFileEx(hFile, buff, 4096, &ov, foo); // 종료 후 foo를 호출해달라

    if(ret == TRUE)
    {
        printf("Start\n");
    }

    SleepEx(10000, TRUE);
 
    CloseHandle(hFile);
    _getch();
}
```

---

## 비동기 I/O 결과 얻기

```cpp
#include <Windows.h>
#include <stdio.h>
#include <conio.h>

int main()
{
    HANDLE hFile = CreateFileA("C:\\Windows\\system32\\calc.exe", GENERIC_READ, FILE_SHARE_READ, 0, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, 0);

    if(hFile == INVALID_HANDLE_VALUE)
    {
        printf("Error in Open File\n");
    }

    char buff[4096] = {0};
    OVERLAPPED ov = { 0 };
    ov.hEvent = CreateEvent(0, 0, 0, 0);

    BOOL ret = ReadFile(hFile, buff, 4096, 0, &ov);

    if(ret == FALSE)
    {
        if(GetLastError() == ERROR_IO_PENNDING)
            printf("Read Error\n");

        WaitForSingleObject(ov.hEvent, INFINITE);

        // 결과 조회
        // ov.Internal  -> error Code
        // ov.InternalHigh -> I/O 작업의 데이터 크기
        printf("Error : %d\n", ov.Internal);
        printf("Bytes : %d\n", ov.InternalHigh);

        // 이렇게 해도 바이트 확인 가능
        DWORD bytes = 0;
        GetOverlappedResult(hFile, &ov, bytes, TRUE);
    }
    else
    {
        printf("Comleting writes synchronosly\n");
    }

    CloseHandle(hFile);
    _getch();
}
```

## 비동기 I/O주의사항

```cpp
int main()
{
    HANDLE hFile = CreateFileA("C:\\Windows\\system32\\calc.exe", GENERIC_READ, FILE_SHARE_READ, 0, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, 0);

    if(hFile == INVALID_HANDLE_VALUE)
    {
        printf("Error in Open File\n");
    }

    {
        char buff[4096] = {0};
        OVERLAPPED ov = { 0 };
        ov.hEvent = CreateEvent(0, 0, 0, 0);

        BOOL ret = ReadFile(hFile, buff, 4096, 0, &ov);
    }
    
    // 여기로 올경우 buff, ov는 파괴된다.
    // ReadFile은 비동기로 동작중이니 잘못된 위치에 데이터를 쓰게 된다. 주의!
    
```