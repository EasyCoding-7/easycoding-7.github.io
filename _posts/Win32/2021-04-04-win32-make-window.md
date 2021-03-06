---
layout: post
title:  "(Win32) make window"
summary: ""
author: win32
date: '2021-04-04 0:00:00 +0000'
category: ['win32']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/make-window/
---

* [Get Code](https://github.com/EasyCoding-7/win32-example/tree/master/2)

---

## 우선 WinMain의 매개변수에 대해 분석해 보자

```cpp
int CALLBACK WinMain(
	HINSTANCE hInstance,
	HINSTANCE hPrevInstance,
	LPSTR lpCmdLine,
	int nCmdShow)
{
	while (true);

	return 0;
}
```

* hInstance : 프로그램의 인스턴스 핸들(현 윈도우의 핸들이라 받아들이자)입니다.
* hPrevInstance : 바로 앞에 실행된 현재 프로그램의 인스턴트 핸들입니다. 없을 경우 NULL이 되며 Win32에서는 항상 NULL 값을 가집니다. **현재는 사용이 안되지만** 16비트와의 호환성을 위해 존재합니다.
* lpCmdLine : 명령행으로 입력된 프로그램 인수입니다. 도스의 argv인수에 해당하며 보통 실행 직후에 파일 경로를 전달합니다.
* nCmdShow :프로그램이 실행될 형태입니다. 최소화, 보통, 최대화 등을 전달합니다.

* CALLBACK : 윈도우에 의해서 호출을 당함을 의미
    * __cdecl : 실행 파일 크기 *크다* -> 호출자에 코드를 작성
    * __stdcall : 실행 파일 크기 *작다* -> 피호출자에 코드를 작성

![](/assets/img/posts/win32/dxd-basic-2-1.png){:class="img-fluid"}

* [참고사이트](http://www.tipssoft.com/bulletin/board.php?bo_table=FAQ&wr_id=625)

---

## 윈도우를 만들어보자

```cpp
#include <Windows.h>

int CALLBACK WinMain(
	HINSTANCE hInstance,
	HINSTANCE hPrevInstance,
	LPSTR lpCmdLine,
	int nCmdShow)
{
	const auto pClassName = "hw3dbutts";
	// register window class
	// 자세한 설명은 검색 추천 ... 일단 받아들인다.
	WNDCLASSEX wc = {0};
	wc.cbSize = sizeof(wc);
	wc.style = CS_OWNDC;
	wc.lpfnWndProc = DefWindowProc;
	wc.cbClsExtra = 0;
	wc.cbWndExtra = 0;
	wc.hInstance = hInstance;
	wc.hIcon = nullptr;
	wc.hCursor = nullptr;
	wc.hbrBackground = nullptr;
	wc.lpszMenuName = nullptr;
	wc.lpszClassName = pClassName;
	wc.hIconSm = nullptr;
	RegisterClassEx(&wc);

	// create window
	// 역시 자세한 설명은 검색 추천
	HWND hWnd = CreateWindowEx(
		0, pClassName,
		"Happy Hard Window",
		WS_CAPTION | WS_MINIMIZEBOX | WS_SYSMENU,
		200, 200, 640, 480,
		nullptr, nullptr, hInstance, nullptr
		);

	ShowWindow(hWnd, SW_SHOW);
	
	while (true);

	return 0;
}
```

여기까지 실행하면 아래와 같이 실행된다.

![](/assets/img/posts/win32/dxd-basic-2-2.png){:class="img-fluid"}

---

## 참고 : 인스턴스 등

* [참고사이트](https://junk-s.tistory.com/49)

* HINSTANCE : 

프로그램의 HANDLE을 의미한다. 사용자가 만드는 것이 아니라 프로그램 시작할 때 운영체제가 제공해준다.<br>
윈도우 운영체제에서 실행되는 프로그램들을 구별하기 위한 ID값을 의미 한다.<br>
window Handle 과 instance는 백업을 하고 사용한다.<br>
HINSTANCE 핸들은 보통 실행되고 있는 Win32 프로그램이 메모리 상에 올라가 있는 시작 주소 값을 갖고 잇습니다.<br>
wWinMain()에서 한 번 들어오는 값으로 값을 저장해서 사용합니다.<br>

* instance

객체 지향 프로그래밍(OOP)에서 인스턴스(instance)는 해당 클래스의 구조로 컴퓨터 저장공간에서 할당된 실체를 의미한다. <br>
즉, 메모리에 올라가 있는 실체를 의미한다. <br>
프로그램의 시작 주소값을 의미한다. <br>

