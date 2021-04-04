---
layout: post
title:  "(Win32) WM_CHAR / mouse event"
summary: ""
author: win32
date: '2021-04-04 0:00:00 +0000'
category: ['win32']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/wmchar-mouse/
---

* [Get Code](https://github.com/EasyCoding-7/win32-example/tree/master/5)

---

## 키보드 입력받을시 탑바 수정

```cpp
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
	case WM_CHAR: 
	{
		static std::string title;
		title.push_back((char)wParam);
		SetWindowText(hWnd, title.c_str());
		break;
	}
	}
	return DefWindowProc(hWnd, msg, wParam, lParam);
}
```

![](/assets/img/posts/win32/dxd-basic-5-1.png){:class="img-fluid"}

---

## 마우스 포인터 받을시 탑바 수정

```cpp
#include <Windows.h>
#include "WindowsMessageMap.h"
#include <sstream>

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
	case WM_CHAR: 
	{
		static std::string title;
		title.push_back((char)wParam);
		SetWindowText(hWnd, title.c_str());
		break;
	}
	case WM_LBUTTONDOWN:
	{
		const POINTS pt = MAKEPOINTS(lParam);
		std::ostringstream oss;
		oss << "(" << pt.x << "," << pt.y << ")";
		SetWindowText(hWnd, oss.str().c_str());
		break;
	}
	}
	return DefWindowProc(hWnd, msg, wParam, lParam);
}
```

![](/assets/img/posts/win32/dxd-basic-5-2.png){:class="img-fluid"}