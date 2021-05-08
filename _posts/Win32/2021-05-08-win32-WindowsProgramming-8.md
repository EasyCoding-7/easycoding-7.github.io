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