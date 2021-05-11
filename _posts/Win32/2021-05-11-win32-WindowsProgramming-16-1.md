---
layout: post
title:  "(Win32 : WindowsProgramming-16-1) Kernel Object 상속"
summary: ""
author: win32
date: '2021-05-11 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-16-1/
---

```cpp
// 데스크톱 어플리케이션 기반으로 생성

#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

LRESULT __stdcall WndProc(HWND hwnd UINT message, UINT message, WPARAM wParam, LPARAM lParam)
{
    static HWND hEdit;

    switch(message)
    {
    case WM_CREATE:
        // 자식 윈도우로 에디트 박스 하나 생성
        hEdit = CreateWindowEx(0, _T("edit"), _T(""), 
                ES_MULTILINE | WS_CHILD | WS_VISIBLE | WS_BORDER,
                10, 10, 300, 500, hwnd, (HMENU)1, 0, 0);
        break;
    case WM_DESTROY:
        PostQuitMessage(0);
        break;
    }
    return DefWindowProc(hwnd, message, wParam, lParam);
}

// ...
```

마우스를 클릭시 핑을 보내는 프로그램을 만들어 보자

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

LRESULT __stdcall WndProc(HWND hwnd UINT message, UINT message, WPARAM wParam, LPARAM lParam)
{
    static HWND hEdit;

    switch(message)
    {
    case WM_CREATE:
        hEdit = CreateWindowEx(0, _T("edit"), _T(""), 
                ES_MULTILINE | WS_CHILD | WS_VISIBLE | WS_BORDER,
                10, 10, 300, 500, hwnd, (HMENU)1, 0, 0);
        break;
    case WM_LBUTTONDOWN:
        STARTUPINFO si = { 0 };
        si.cb = sizeof(si);
        PROCESS_INFORMATION pi = { 0 };
        TCHAR name[] = _T("ping www.microsoft.com");

        // CreateProcess를 통해서 ping.exe를 실행
        BOOL b = CreatePrcess(0, name, 0, 0, FALSE, 0, 0, 0, &si, &pi);

        if(b)
        {
            CloseHandle(pi.hProcess);
            CloseHandle(pi.hTread);
        }
        break;
    case WM_DESTROY:
        PostQuitMessage(0);
        break;
    }
    return DefWindowProc(hwnd, message, wParam, lParam);
}

// ...
```

ping은 콘솔 어플리케이션인데 a.txt에 출력을 하게 변경해보자.(표준 입출력 Redirect라 한다.)

```cpp
case WM_LBUTTONDOWN:
    // 파일 생성
    HANDLE hFile = CreateFile(_T("a.txt"), GENERIC_READ | GENERIC_WRITE,
                    FILE_SHARE_READ | FILE_SHARE_WRITE, 0, CREATE_ALWAYS,
                    FILE_ATTRIBUTE_NORMAL, 0);

    STARTUPINFO si = { 0 };
    si.cb = sizeof(si);
    // Redirection 추가
    si.hStdOutput = hFile;      // 이렇게 주면 자식 프로세스 기준에서 hFile(HANDLE)의 내용을 모름
    si.dwFlags = STARTF_USESTDHANDLES;

    PROCESS_INFORMATION pi = { 0 };
    TCHAR name[] = _T("ping www.microsoft.com");

    BOOL b = CreatePrcess(0, name, 0, 0, FALSE, 0, 0, 0, &si, &pi);

    if(b)
    {
        CloseHandle(pi.hProcess);
        CloseHandle(pi.hTread);
    }
    break;
```

부모 Process가 자신이 사용하던 HANDLE을 상속 가능

```cpp
case WM_LBUTTONDOWN:
    HANDLE hFile = CreateFile(_T("a.txt"), GENERIC_READ | GENERIC_WRITE,
                    FILE_SHARE_READ | FILE_SHARE_WRITE, 0, CREATE_ALWAYS,
                    FILE_ATTRIBUTE_NORMAL, 0);

    // hFile자체도 상속이 가능하게 변경해야 한다.
    SetHandleInformation(hFile, HANDLE_FLAG_INGERIT, HANDLE_FLAG_INGERIT);

    STARTUPINFO si = { 0 };
    si.cb = sizeof(si);
    si.hStdOutput = hFile;
    si.dwFlags = STARTF_USESTDHANDLES;

    PROCESS_INFORMATION pi = { 0 };
    TCHAR name[] = _T("ping www.microsoft.com");

    BOOL b = CreatePrcess(0, name, 0, 0, TRUE, // FALSE -> TRUE 변경 (부모의 Kernel Object를 상속 받겠다.)
                        0, 0, 0, &si, &pi);
```

또 다른 방법이 있음, Kernel Object를 생성 시 SECURITY_ATTRIBUTES를 전달한다.

```cpp
SECURITY_ATTRIBUTES sa = { 0 };
sa.bInheritHandle = TRUE;       // 상속가능여부
sa.lpSecurityDescriptor = 0;
sa.nLength = sizeof(sa);

HANDLE hFile = CreateFile(_T("b.txt"), GENERIC_READ | GENERIC_WRITE,
                FILE_SHARE_READ | FILE_SHARE_WRITE, &sa, CREATE_ALWAYS,
                FILE_ATTRIBUTE_NORMAL, 0);
```

---

a.txt에 나오는 출력결과를 edit box로 옮겨보자

```cpp
#undef UNICODE
#undef _UNICODE

// ..

case WM_LBUTTONDOWN:
    HANDLE hFile = CreateFile(_T("a.txt"), GENERIC_READ | GENERIC_WRITE,
                    FILE_SHARE_READ | FILE_SHARE_WRITE, 0, CREATE_ALWAYS,
                    FILE_ATTRIBUTE_NORMAL, 0);

    SetHandleInformation(hFile, HANDLE_FLAG_INGERIT, HANDLE_FLAG_INGERIT);

    STARTUPINFO si = { 0 };
    si.cb = sizeof(si);
    si.hStdOutput = hFile;
    si.dwFlags = STARTF_USESTDHANDLES;

    PROCESS_INFORMATION pi = { 0 };
    TCHAR name[] = _T("ping www.microsoft.com");

    BOOL b = CreatePrcess(0, name, 0, 0, TRUE, CREATE_NO_WINDOW // 콘솔창 만들지 말아 달라
                          , 0, 0, &si, &pi);

    if(b)
    {
        WaitForSingleObject(pi.hProcess, INFINITE);

        // 파일 포인터를 제일 앞으로 옮긴다.
        SetFilePointer(hFile, 0, 0, FILE_BEGIN);

        DWORD len;
        char buff[4096] = { 0 };

        // 파일에서 읽어내고
        BOOL b1 = ReadFile(hFile, buff, 4096, &len, 0);

        // buff의 내용을 edit에 출력
        SendMessage(hEdit, EM_REPLACESEL, 0, (LPARAM)buff);

        CloseHandle(pi.hProcess);
        CloseHandle(pi.hTread);
    }
    break;
```

ping의 출력을 줄 단위로 읽어오게 하자 -> 파일로 쓰지말고 pipe를 쓰자

```cpp
case WM_LBUTTONDOWN:
    HANDLE hRead, hWrite;

    CreatePipe(&hRead, &hWrite, 0, 1024);

    SetHandleInformation(hWrite, HANDLE_FLAG_INGERIT, HANDLE_FLAG_INGERIT);

    STARTUPINFO si = { 0 };
    si.cb = sizeof(si);
    si.hStdOutput = hWrite;
    si.dwFlags = STARTF_USESTDHANDLES;

    PROCESS_INFORMATION pi = { 0 };
    TCHAR name[] = _T("ping www.microsoft.com");

    BOOL b = CreatePrcess(0, name, 0, 0, TRUE, CREATE_NO_WINDOW, 0, 0, &si, &pi);

    if(b)
    {
        CloseHandle(hWrite);    // 쓰기 핸들은 이제 필요없기에

        while(1)
        {
            DWORD len;
            char buff[4096] = { 0 };

            BOOL b1 = ReadFile(hRead, buff, 4096, &len, 0);

            if(len <= 0) // pipe가 끊어졌음, 핑이 끝남
                break;

            SendMessage(hEdit, EM_REPLACESEL, 0, (LPARAM)buff);
        }


        CloseHandle(pi.hProcess);
        CloseHandle(pi.hTread);
    }
    break;
```