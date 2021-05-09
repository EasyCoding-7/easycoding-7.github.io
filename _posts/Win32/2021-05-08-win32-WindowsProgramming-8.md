---
layout: post
title:  "(Win32 : WindowsProgramming-8) GUI Programming"
summary: ""
author: win32
date: '2021-05-08 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-8/
---

## Window 만들기

* 윈도우를 만드는 순서
    * 윈도우 클래스 만들기
    * 윈도우 클래스 시스템에 등록
    * 등록된 윈도우 클래스를 이용해서 윈도우 만들기
    * 윈도우 나타내기

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

int main()
{
    // 윈도우 클래스 만들기
    WNDCLASSEX wcex = {0};
    wcex.cbSize = sizeof(wcex);
    wcex.lpfnWndProc = DefWindowProc;
    wcex.lpszCalssName = _T("MyWindow");

    // 윈도우 클래스 시스템에 등록
    ATOM atom = RegisterClassEx(&wcex);

    if(atom == 0)
    {
        printf("등록 실패 : %d\n", GetLastError());
    }

    // 등록된 윈도우 클래스를 이용해서 윈도우 만들기
    HWND hwnd = CreateWindowEx(0,
                               _T("MyWindow"),         // 클래스 이름
                               _T("Hello"),            // 캡션 문자열
                               WS_OVERLAPPEDWINDOW,    // 윈도우 스타일
                               0, 0, 500, 500,         // 윈도우 위치
                               0, 0, 0, 0);

    // 윈도우 나타내기
    ShowWindow(hwnd, SW_SHOW);


    MessageBox(0, _T(""), _T(""), MB_OK);   // 프로그램이 종료되지 않게 막기위해서 넣음
}
```

---

## 문제점 해결

* 사이즈 조절 후 무슨색으로 배경색을 채워야 하는지 안넣음
* 커서가 변경될때 어떻게 변경될지 안넣음

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

int main()
{
    WNDCLASSEX wcex = {0};
    wcex.cbSize = sizeof(wcex);
    wcex.lpfnWndProc = DefWindowProc;
    wcex.lpszCalssName = _T("MyWindow");
    wcex.hbrBackground = CreateSolidBrush(RGB(255, 255, 255));  // 배경색
    wcex.hCursor = LoadCursor(0, IDC_ARROW);        // 커서

    ATOM atom = RegisterClassEx(&wcex);

    if(atom == 0)
    {
        printf("등록 실패 : %d\n", GetLastError());
    }

    HWND hwnd = CreateWindowEx(0,
                               _T("MyWindow"),
                               _T("Hello"),
                               WS_OVERLAPPEDWINDOW,
                               0, 0, 500, 500,
                               0, 0, 0, 0);

    ShowWindow(hwnd, SW_SHOW);


    MessageBox(0, _T(""), _T(""), MB_OK);
}
```

최소화 버튼을 비활성화

```cpp
    HWND hwnd = CreateWindowEx(0,
                               _T("MyWindow"),
                               _T("Hello"),
                               WS_OVERLAPPEDWINDOW & ~WS_MINIMIZEBOX,
                               0, 0, 500, 500,
                               0, 0, 0, 0);
```

확장스타일 : TOPMOST 주기

```cpp
    HWND hwnd = CreateWindowEx(WS_EX_TOPMOST,
                               _T("MyWindow"),
                               _T("Hello"),
                               WS_OVERLAPPEDWINDOW & ~WS_MINIMIZEBOX,
                               0, 0, 500, 500,
                               0, 0, 0, 0);
```

---

## 윈도우 프로시저

`CreateWindowEx`가 호출 될 시 윈도우는 **메시지 큐**를 생성한다.<br>
메시지 큐는 GUI에서 발생한 이벤트를 담고있고, 그 처리를 `GetMessage`를 통해서 한다.

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

int main()
{
    WNDCLASSEX wcex = {0};
    wcex.cbSize = sizeof(wcex);
    wcex.lpfnWndProc = DefWindowProc;
    wcex.lpszCalssName = _T("MyWindow");
    wcex.hbrBackground = CreateSolidBrush(RGB(255, 255, 255));  // 배경색
    wcex.hCursor = LoadCursor(0, IDC_ARROW);        // 커서

    ATOM atom = RegisterClassEx(&wcex);

    HWND hwnd = CreateWindowEx(0,
                               _T("MyWindow"),
                               _T("Hello"),
                               WS_OVERLAPPEDWINDOW,
                               0, 0, 500, 500,
                               0, 0, 0, 0);

    ShowWindow(hwnd, SW_SHOW);

    // 메시지 처리(메시지 루프)
    MSG msg;
    while(1)
    {
        GetMessage(&msg, hwnd, 0, 0);

        if(msg.mesage == WM_LBUTTONDOWN)
        {
            printf("LBUTTONDOWN\n");
        }
    }
}
```

이렇게 할 경우 윈도우가 움직이거나 사이즈 조절이 안된다.

```cpp
    MSG msg;
    while(1)
    {
        GetMessage(&msg, hwnd, 0, 0);

        if(msg.mesage == WM_LBUTTONDOWN)
        {
            printf("LBUTTONDOWN\n");
        }
        else
        {
            // 내가 처리하지 않은 메시지를 디폴트 처리기에 넘긴다.
            DefWindowProc(msg.hwnd, msg.message, msg.wParam, msg.lParam);
        }
    }
```

## 메시지를 처리하기 위한 함수 윈도우 프로시저(Window Procedure)

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

int main()
{
    WNDCLASSEX wcex = {0};
    wcex.cbSize = sizeof(wcex);
    wcex.lpfnWndProc = DefWindowProc;
    wcex.lpszCalssName = _T("MyWindow");
    wcex.hbrBackground = CreateSolidBrush(RGB(255, 255, 255));  // 배경색
    wcex.hCursor = LoadCursor(0, IDC_ARROW);        // 커서

    ATOM atom = RegisterClassEx(&wcex);

    HWND hwnd = CreateWindowEx(0,
                               _T("MyWindow"),
                               _T("Hello"),
                               WS_OVERLAPPEDWINDOW,
                               0, 0, 500, 500,
                               0, 0, 0, 0);

    ShowWindow(hwnd, SW_SHOW);

    MSG msg;
    while(1)
    {
        GetMessage(&msg, hwnd, 0, 0);

        // 여기에서 모든 메시지를 처리하면 코드의 길이가 너무 길어진다.
        if(msg.mesage == WM_LBUTTONDOWN)
        {
            printf("LBUTTONDOWN\n");
        }
        else
        {
            DefWindowProc(msg.hwnd, msg.message, msg.wParam, msg.lParam);
        }
    }
}
```

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

LRESULT __stdcall WndProc(HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    switch(message)
    {
        case WM_LBUTTONDOWN:
            printf("LBUTTONDOWN\n");
            break;
        case WM_KEYDOWN:
            printf("LBUTTONDOWN\n");
            break;
    }

    return DefWindowProc(msg.hwnd, msg.message, msg.wParam, msg.lParam);
}

int main()
{
    WNDCLASSEX wcex = {0};
    wcex.cbSize = sizeof(wcex);
    wcex.lpfnWndProc = WndProc;             // 메시지를 누가 처리할지 알림
    wcex.lpszCalssName = _T("MyWindow");
    wcex.hbrBackground = CreateSolidBrush(RGB(255, 255, 255));
    wcex.hCursor = LoadCursor(0, IDC_ARROW);

    ATOM atom = RegisterClassEx(&wcex);

    HWND hwnd = CreateWindowEx(0,
                               _T("MyWindow"),
                               _T("Hello"),
                               WS_OVERLAPPEDWINDOW,
                               0, 0, 500, 500,
                               0, 0, 0, 0);

    ShowWindow(hwnd, SW_SHOW);

    MSG msg;
    while(1)
    {
        GetMessage(&msg, hwnd, 0, 0);

        DispatchMessage(&msg);
    }
}
```

문제) 현재는 프로그램이 죽지 않고 무한으로 돈다.

---

## 종료하는 과정을 추가

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

LRESULT __stdcall WndProc(HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    switch(message)
    {
        case WM_LBUTTONDOWN:
            printf("LBUTTONDOWN\n");
            break;
        case WM_KEYDOWN:
            printf("LBUTTONDOWN\n");
            break;
        case WM_DESTROY:
            // exit(0);        // 이건 강제 종료
            // 아래 while(1) 문 밑에 더 처리할 부분이 있었다면 그 코드는 돌지 않는다.
            PostQuitMessage(0); // 루프를 종료해 달라
            break;
    }

    return DefWindowProc(msg.hwnd, msg.message, msg.wParam, msg.lParam);
}

int main()
{
    WNDCLASSEX wcex = {0};
    wcex.cbSize = sizeof(wcex);
    wcex.lpfnWndProc = WndProc;             // 메시지를 누가 처리할지 알림
    wcex.lpszCalssName = _T("MyWindow");
    wcex.hbrBackground = CreateSolidBrush(RGB(255, 255, 255));
    wcex.hCursor = LoadCursor(0, IDC_ARROW);

    ATOM atom = RegisterClassEx(&wcex);

    HWND hwnd = CreateWindowEx(0,
                               _T("MyWindow"),
                               _T("Hello"),
                               WS_OVERLAPPEDWINDOW,
                               0, 0, 500, 500,
                               0, 0, 0, 0);

    ShowWindow(hwnd, SW_SHOW);

    MSG msg;
    while(1)
    {
        GetMessage(&msg, hwnd, 0, 0);

        if(msg.message == WM_QUIT)
            break;
        DispatchMessage(&msg);
    }
}
```

또 다른 방법?

```cpp
    MSG msg;
    while(GetMessage(&msg, hwnd, 0, 0))
    {
        // WM_QUIT이 GetMessage로 들어오면 false가 리턴된다.
        DispatchMessage(&msg);
    }
```

---

## 응용 프로그램의 종류

* 콘솔 어플리케이션
    * entry : main
    * 콘솔창 : 자동 생성
    * 링커 옵션 : `/subsystem:console`
* 데스크탑 어플리케이션
    * entry : WinMain
    * 콘솔창 : 생성 안됨
    * 링커 옵션 : `/subsystem:windows`

결국 링커 옵션만 변경하면 둘은 동일하다

```cpp
// 이렇게 코드상에서 링커를 변경할 수도 있다.
#pragma comment(linker, "subsystem:console");
```