---
layout: post
title:  "(Win32) Window message"
summary: ""
author: win32
date: '2021-04-04 0:00:00 +0000'
category: ['win32']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/window-message/
---

* [Get Code](https://github.com/EasyCoding-7/win32-example/tree/master/4)

---

## 들어오는 위도우 메시지를 모두 출력해보자

`WindowsMessageMap.h` 는 코드 참조

```cpp
#include <Windows.h>
#include "WindowsMessageMap.h"

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
	static WindowsMessageMap mm;
	OutputDebugString(mm(msg, lParam, wParam).c_str());

	switch (msg)
	{
	case WM_CLOSE:
		PostQuitMessage(69);
		break;
	}
	return DefWindowProc(hWnd, msg, wParam, lParam);
}
```

![](/assets/img/posts/win32/dxd-basic-4-1.png){:class="img-fluid"}

---

## 메시지 처리해보기

```cpp
#include <Windows.h>
#include "WindowsMessageMap.h"

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
	static WindowsMessageMap mm;
	OutputDebugString(mm(msg, lParam, wParam).c_str());

	switch (msg)
	{
	case WM_CLOSE:
		PostQuitMessage(69);
		break;
	case WM_KEYDOWN:
		if (wParam == 'F')
		{
			SetWindowText(hWnd, "Respects");
		}
		break;
	}
	return DefWindowProc(hWnd, msg, wParam, lParam);
}
```