---
layout: post
title:  "(Win32 : WindowsProgramming-30) IOCP"
summary: ""
author: win32
date: '2021-05-16 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-30/
---
 
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
        {
            WaitForSingleObject(ov.hEvent, INFINITE);
            // 현재 문제는 비동기 I/O를 요청한 스레드와 대기하는 스레드가 동일 스레드이다.
            // 대기를 다른 스레드로 분리하자
        }
    }
    else
    {
        printf("Read Error\n");
    }

    CloseHandle(hFile);
    _getch();
}
```

```cpp
#include <Windows.h>
#include <stdio.h>
#include <conio.h>

DWORD __stdcall foo(void* p)
{
    OVERLAPPED* pOv = (OVERLAPPED*)p;

    WaitForSingleObject(pOv->hEvent, INFINITE);

    printf("foo : I/O Complete\n");
    printf("error : %d\n", pOv->Internal);
    printf("bytes : %d\n", pOv->InternalHigh);

    // 그런데 복사된 데이터는 buff에 있는데 이 Thread에서는 접근이 불가능하다?

    return 0;
}

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
        {
            CreateThread(0, 0, foo, &ov, 0, 0);
        }
    }
    else
    {
        printf("Read Error\n");
    }

    CloseHandle(hFile);
    _getch();
}
```

```cpp
#include <Windows.h>
#include <stdio.h>
#include <conio.h>

struct OVERLAPPED_PLUS : public OVERLAPPED
{
    HANDLE hFile;
    char* buff;
};

DWORD __stdcall foo(void* p)
{
    OVERLAPPED_PLUS* pOv = (OVERLAPPED_PLUS*)p;

    WaitForSingleObject(pOv->hEvent, INFINITE);

    printf("foo : I/O Complete\n");
    printf("error : %d\n", pOv->Internal);
    printf("bytes : %d\n", pOv->InternalHigh);

    // 버퍼사용
    // ...

    delete pOv->buff;   // 버퍼제거
    CloseHandle(pOv->hFile);

    return 0;
}

int main()
{
    HANDLE hFile = CreateFileA("C:\\Windows\\system32\\calc.exe", GENERIC_READ, FILE_SHARE_READ, 0, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, 0);

    if(hFile == INVALID_HANDLE_VALUE)
    {
        printf("Error in Open File\n");
    }

    char buff[4096] = {0};
    OVERLAPPED_PLUS ov = new OVERLAPPED_PLUS;
    memset(ov, 0, sizeof(OVERLAPPED_PLUS));
    ov->hFile = hFile;
    op->buff = new char[4096];
    ov->hEvent = CreateEvent(0, 0, 0, 0);

    BOOL ret = ReadFile(hFile, ov->buff, 4096, 0, ov);

    if(ret == FALSE)
    {
        if(GetLastError() == ERROR_IO_PENNDING)
        {
            CreateThread(0, 0, foo, ov, 0, 0);
        }
    }
    else
    {
        printf("Read Error\n");
    }

    _getch();
}
```

이런 작업을 윈도우에서 좀 더 쉽게 할 수 있게 제공해준다 -> **IOCP**

---

## 입출력 완료 포트(I/O Completion Port)

* 수백개의 비동기 I/O 작업을 효율적으로 처리하기 위한 기술
* 대용량 서버 제작을 위한 기술

```cpp
#include <Windows.h>
#include <stdio.h>
#include <conio.h>

struct OVERLAPPED_PLUS : public OVERLAPPED
{
    int id;
    HANDLE hFile;
    char* buff;
    OVERLAPPED_PLUS() { memset(this, 0, sizeof(OVERLAPPED_PLUS)); }
};

BOOL Read4kAsync(HANDLE hFile, int offset, int id)
{
    OVERLAPPED_PLUS* op = new OVERLAPPED_PLUS;
    op->id = id;
    op->hFile = hFile;
    op->buff = new char[4096];
    op->hEvent = CreateEvent(0, 0, 0, 0);
    return ReadFile(op->hFile, op->buff, 4096, 0, op);
}

int main()
{
    HANDLE hFile1 = CreateFileA("C:\\Windows\\system32\\calc.exe", GENERIC_READ, FILE_SHARE_READ, 0, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, 0);

    HANDLE hFile2 = CreateFileA("C:\\Windows\\system32\\notepad.exe", GENERIC_READ, FILE_SHARE_READ, 0, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, 0);

    Read4kAsync(hFile1, 0, 1);
    Read4kAsync(hFile1, 4096, 2);
    Read4kAsync(hFile2, 0, 3);
    Read4kAsync(hFile2, 4096, 4);

    printf("Main Continue .. \n");
    _getch();
}
```

IOCP를 이용하여 위 코드를 효율화 한다.

```cpp
DWORD __stdcall foo(void* p)
{
    HANDLE hIOCP = (HANDLE)p;

    DWORD bytes = 0
    DWORD error = 0;
    DWORD key = 0;
    OVERLAPPED_PLUS* op = 0;

    while(GetQueuedCompletionStatus(hIOCP, &bytes, &key, (OVERLAPPED_PLUS**)&op, INFINITE))
    {
        printf("COMPLETE : %d, %d, %d\n", key, op->id, bytes);

        // ...

        delete op->buff;
        delete op;
    }
    return 0;
}

int main()
{
    // 1. IOCP 생성
    HANDLE hIOCP = CreateIoCompletionPort(INVALID_HANDLE_VALUE, 0, 0, 2/*IOCP를 위해 몇개의 스레드를 사용할 것인가*/);

    HANDLE hFile1 = CreateFileA("C:\\Windows\\system32\\calc.exe", GENERIC_READ, FILE_SHARE_READ, 0, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, 0);

    HANDLE hFile2 = CreateFileA("C:\\Windows\\system32\\notepad.exe", GENERIC_READ, FILE_SHARE_READ, 0, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, 0);

    // 2. IOCP에 장치 등록
    CreateIoCompletionPort(hFile1, hIOCP, 100/*완료키*/, 2);
    CreateIoCompletionPort(hFile2, hIOCP, 200, 2);

    // 3. IOCP에서 완료 큐를 대기할 스레드 생성
    HANDLE hThread1 = CreateThread(0, 0, foo, (void*)hIOCP, 0, 0);
    HANDLE hThread2 = CreateThread(0, 0, foo, (void*)hIOCP, 0, 0);

    // 비동기 요청
    Read4kAsync(hFile1, 0, 1);
    Read4kAsync(hFile1, 4096, 2);
    Read4kAsync(hFile2, 0, 3);
    Read4kAsync(hFile2, 4096, 4);

    printf("Main Continue .. \n");
    _getch();
}
```

---

## IOCP를 이용한 NetworkSystem Example

* TCP서버를 만들고
* Client의 접속을 대기하다
* Client가 접속 후 Data를 수신한다.
* 단, Client가 어떻게 들어올지 모르니 Client의 접속/Data수신은 비동기식으로 처리한다.

```cpp
#define WIN32_LEAN_AND_MEAN
#include <stdio.h>
#include <Windows.h>
#include <WinSock2.h>
#include <stdlib.h>
#pragma comment(lib, "ws2_32.lib")

struct OVERLAPPED_PLUS : public OVERLAPPED
{
    int sock;       // sock 번호
    WSABUF wsaBuf;  // 버퍼
};

DWORD __stdcall EchoThread(void* p)
{
    HANDLE hPort = (HANDLE)p;
    WSAOVERLAPPED* pov;
    DWORD bytes, key;

    while(1)
    {
        GetQueuedCompletionStatus(hPort, &bytes, &key, &pov, INFINITE);

        OVERLAPPED_PLUS* op = (OVERLAPPED_PLUS*)pov;

        printf("수신된 data : %s\n", op->wsaBuf.buf);

        closesocket(op->sock);
        delete op->wsaBuf.buf;
        CloseHandle(op->hEvent);
        delete op;
    }
    return 0;
}

int main()
{
    WSADATA w;
    int ret = WSAStartup(MAKEWORD(2, 2), &w);

    // 1. IOCP 생성
    HANDLE hPort = CreateIoCompletionPort(INVALID_HANDLE_VALUE, 0, 0, 2);

    // 2. IOCP에서 대기할 스레드 생성
    HANDLE h1 = CreateThread(0, 0, EchoThread (void*)hPort, 0, 0);
    HANDLE h2 = CreateThread(0, 0, EchoThread (void*)hPort, 0, 0);

    // Listen 소캣
    int listen_sock = WSASocket(PF_INET, SOCK_STREAM, 0, 0, 0, WSA_FLAG_OVERLAPPED);
    SOCKADDR_IN addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(4000);
    addr.sin_addr.s_addr = INADDR_ANY;

    bind(listen_sock, (struct sockaddr*)&addr, sizeof addr);
    listen(listen_sock, 5);

    int cnt = 0;
    while(1)
    {
        struct sockaddr_in addr2;
        int sz = sizeof addr2;

        // Client 대기
        int link_sock = accept(listen_sock, (struct sockaddr*)&addr2, &sz)

        CreateIoCompletionPort((HANDLE)link_sock, hPort, link_sock, 2);

        printf("클라이언트 접속");

        // IO 작업당 아래 구조체를 만들어야 한다.
        OVERLAPPED_PLUS* op = new OVERLAPPED_PLUS;
        op->wsaBuf.buf = new char[1024];
        op->wsaBuf.len = 1024;
        op->hEvent = WSACreateEvent();

        DWORD flag = 0;
        DWORD recvBytes = 0

        int n = WSARecv(link_sock, 
                        &(op->wsaBuf), 1,   // 버퍼와 버퍼 개수
                        &recvBytes,         // 받은 data 크기
                        &flag,
                        op,                 // overlapped구조체
                        0);
    }
    WSACleanup();
}
```