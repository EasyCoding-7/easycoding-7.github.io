---
layout: post
title:  "(Win32 : WindowsProgramming-9) 윈도우 핸들과 API"
summary: ""
author: win32
date: '2021-05-09 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-9/
---

## 핸들이란?

윈도우(OS)에서 각 객체에 부여한 고유한 번호(정수)

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

int main()
{
    // 메모장의 핸들을 받는다.
    HWND hwnd = FindWindow(0, _T("제목 없음 - 메모장"));
    printf("%d\n", hwnd);

    // 특정 포인트에 있는 윈도우의 핸들을 받는다.
    POINT pt = {10, 10};
    HWND hwnd2 = WindowFromPoint(pt);

    // 메모장을 다른 프로세스에서 컨트롤할 수 있다.
    getchar(); MoveWindow(hwnd, 0, 0, 300, 300, TRUE);
    getchar(); ShowWindow(hwnd, SW_HIDE);
    getchar(); ShowWindow(hwnd, SW_SHOW);

    // 단, 다른 스레드에서 사용된 DestroyWindow는 해당 hwnd을 종료하지 못함
    DestroyWindow(hwnd);

    TCHAR name[256];
    GetClassName(hwnd, name, 256);  // 클래스 이름을 받을 수 있다.

    //printf("%s\n", name);         // 유니코드 주의
    _tprintf(_T("%s\n"), name);
}
```

---

## 핸들 타입의 정체

핸들이 단순 int를 typedef할 경우 HWND, HICON ... 어떤게 들어갈지 알 수 없다.

해결책?

```cpp
struct HWND__
{};
typedef struct HWND__* HWND;

struct HICON__
{};
typedef struct HICON__* HICON;
```

* 모든 핸들 타입은 구조체의 포인터
    * 구조체 자체는 사용하지 안는다.
    * 다른 구조체 포인터 타입은 암시적 형 변환이 되지 않는다.

```cpp
// 단, HANDLE은 모든 핸들을 담을 수 있는데 void*이기 때문이다.
typedef void* HANDLE;
```

---

## 윈도우 Object와 윈도우 스타일

`CreateWindowEx()`를 하면 윈도우 정보(x, y, width, height ...)이 Window Object화 되어 어딘가 저장된다.<br>
저장된 Window Object는 `ShowWindow()`를 통해서 화면에 그려진다.<br>
Window Object에 접근할 방법은 Device Driver 정도를 통해서 접근이 가능하나(힘들다)<br>
단, 스타일 정도는 변경이 가능하다.(`GetWindowLongPtr`, `SetWindowLongPtr`)

```cpp
int _tmain()
{
    HWND hwnd = FindWindow(_T("Notepad"), 0);
    getchar();

    LONG style = GetWindowLongPtr(hwnd, GWL_STYLE);

    style = style & ~WS_CAPTION;

    SetWindowLongPtr(hwnd, GWL_STYLE, style);

    // SetWindowLongPtr호출 후에는 SetWindowPos를 호출해야한다.(Window10 이전버전 기준)
    SetWindowPos(hwnd, 0, 0, 0, 0, 0, SWP_NOZORDER | SWP_NOMOVE | SWP_NOSIZE | SWP_DRAWFRAME);
}
```

---

## 윈도우 메세지

* WPARAM(WORD PARAMETER) : 16비트 시절에 WORD(16bit)라서 WPARAM이라 불림
* LPARAM(LONG PARAMETER) : 말 그대로 LONG(32bit)

메시지를 보내는 방법 2가지

* `PostMessage` : 메시지를 메시지 큐에 넣고 즉시 반환
* `SendMessage` : 메시지를 메시지 큐에 넣고 처리가 완료 될 때까지 대기

다른 프로세스의 윈도우 종료

`SendMessage(hwnd, WM_CLOSE, 0, 0);`

---

## 사용자 정의 메시지

```cpp
#define WM_MYMESSAGE  WM_APP+100

int _tmain()
{
    HWND hwnd = FindWind(0, _T("Hello"));
    getchar();
    SendMessage(hwnd, WM_MYMESSAGE, 0, 0);
}
```