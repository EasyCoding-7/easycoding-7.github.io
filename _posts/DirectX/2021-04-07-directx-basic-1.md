---
layout: post
title:  "(DirectX : Basic) 1. Set Project"
summary: ""
author: DirectX
date: '2021-04-07 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-1/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/1)

**VS2019로 실행할 것.**

---

## 프로젝트 만들기

Windows 데스크톱 애플리케이션

![](/assets/img/posts/directx/basic-1-1.png){:class="img-fluid"}

메시지 루프 구조를 변경한다.

```cpp
    // 기본 메시지 루프입니다:
    while (true)
    {
        // 메시지가 들어올때만 받는게 아니라 있는지를 체크후 꺼내는 형태로 변경
        if (PeekMessage(&msg, nullptr, 0, 0, PM_REMOVE))
        {
            if (msg.message == WM_QUIT)
                break;

            if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
            {
                TranslateMessage(&msg);
                DispatchMessage(&msg);
            }
        }

        // TODO : 게임로직
    }
```

![](/assets/img/posts/directx/basic-1-2.png){:class="img-fluid"}

미리 컴파일된 헤더를 사용하게 변경

```cpp
// pch.h
#pragma once

#include <vector>
#include <memory>
using namespace std;
```

Game class 만들기

```cpp
#pragma once
class Game
{
public:
	void Init();
	void Update();
};
```

```cpp
#include "pch.h"
#include "Game.h"

void Game::Init()
{
}

void Game::Update()
{
}
```

Game을 주기적 호출

```cpp
    MSG msg;

    //Game* game = new Game();
    unique_ptr<Game> game = make_unique<Game>();
    game->Init();

    // 기본 메시지 루프입니다:
    while (true)
    {
        if (PeekMessage(&msg, nullptr, 0, 0, PM_REMOVE))
        {
            if (msg.message == WM_QUIT)
                break;

            if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
            {
                TranslateMessage(&msg);
                DispatchMessage(&msg);
            }
        }

        // TODO : 게임로직
        game->Update();
    }
```

---

## Direct 라이브러리를 관리하는 프로젝트를 하나 더 생성

![](/assets/img/posts/directx/basic-1-3.png){:class="img-fluid"}

정적라이브러리로 생성

* `d3dx12.h`(MS에서 제공하는 라이브러리기능 모음)를 추가
* 미리 컴파일된 헤더에 아래를 추가

```cpp
#pragma once

// 각종 include
#include <windows.h>
#include <tchar.h>
#include <memory>
#include <string>
#include <vector>
#include <array>
#include <list>
#include <map>
using namespace std;

#include "d3dx12.h"
#include <d3d12.h>
#include <wrl.h>
#include <d3dcompiler.h>
#include <dxgi.h>
#include <DirectXMath.h>
#include <DirectXPackedVector.h>
#include <DirectXColors.h>
using namespace DirectX;
using namespace DirectX::PackedVector;
using namespace Microsoft::WRL;

// 각종 lib
#pragma comment(lib, "d3d12")
#pragma comment(lib, "dxgi")
#pragma comment(lib, "dxguid")
#pragma comment(lib, "d3dcompiler")

// 각종 typedef
using int8		= __int8;
using int16		= __int16;
using int32		= __int32;
using int64		= __int64;
using uint8		= unsigned __int8;
using uint16	= unsigned __int16;
using uint32	= unsigned __int32;
using uint64	= unsigned __int64;
using Vec2		= XMFLOAT2;
using Vec3		= XMFLOAT3;
using Vec4		= XMFLOAT4;
using Matrix	= XMMATRIX;

void HelloEngine();
```

---

