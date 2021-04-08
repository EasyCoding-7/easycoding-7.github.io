---
layout: post
title:  "(DirectX : Tutorial) 9. Architecture/SwapChain"
summary: ""
author: DirectX
date: '2021-03-28 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/tutorial-9/
---

* [DirectX Tutorial (YouTube)](https://www.youtube.com/watch?v=_4FArgOX1I4&list=PLqCJpWy5Fohd3S7ICFXwUomYW0Wv67pDD)
* [Get Project(GitHub)](https://github.com/EasyCoding-7/DirectX-basic-Tutorial)

---

## Get Code

* [Link](https://github.com/EasyCoding-7/DirectX-basic-Tutorial/tree/master/9)

---

## DXGI 란?

Direct3D기술은 매 시리즈 변경이 된다.(10, 11, 12 …) 하지만 변경되지 않고 유지되어야할 기술 API집합이 있고 이 집합을 DXGI에 넣어둠(예를들어 IDXGISwapChain(스왑체인))

```cpp
#pragma once
#include "ChiliWin.h"
#include <d3d11.h>

class Graphics
{
public:
	Graphics( HWND hWnd );
	Graphics( const Graphics& ) = delete;
	Graphics& operator=( const Graphics& ) = delete;
	~Graphics();
	void EndFrame();
	void ClearBuffer( float red,float green,float blue ) noexcept;
private:
	ID3D11Device* pDevice = nullptr;            // D3D에 접근하기 위한 포인터
	IDXGISwapChain* pSwap = nullptr;            // SwapChain 포인터
	ID3D11DeviceContext* pContext = nullptr;    // DC(DeviceContext) 포인터
	ID3D11RenderTargetView* pTarget = nullptr;  // 그려지는 버퍼를 읽어오는 포인터
};
```

```cpp
#include "Graphics.h"

#pragma comment(lib,"d3d11.lib")

Graphics::Graphics( HWND hWnd )
{
    // SwapChain에 대한 옵션(필요하다면 찾아볼 것)
	DXGI_SWAP_CHAIN_DESC sd = {};
	sd.BufferDesc.Width = 0;
	sd.BufferDesc.Height = 0;
	sd.BufferDesc.Format = DXGI_FORMAT_B8G8R8A8_UNORM;
	sd.BufferDesc.RefreshRate.Numerator = 0;
	sd.BufferDesc.RefreshRate.Denominator = 0;
	sd.BufferDesc.Scaling = DXGI_MODE_SCALING_UNSPECIFIED;
	sd.BufferDesc.ScanlineOrdering = DXGI_MODE_SCANLINE_ORDER_UNSPECIFIED;
	sd.SampleDesc.Count = 1;
	sd.SampleDesc.Quality = 0;
	sd.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
	sd.BufferCount = 1;
	sd.OutputWindow = hWnd;
	sd.Windowed = TRUE;
	sd.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
	sd.Flags = 0;

	// create device and front/back buffers, and swap chain and rendering context
    // D3D를 생성하며 SwapChain을 연결한다
	D3D11CreateDeviceAndSwapChain(
		nullptr,
		D3D_DRIVER_TYPE_HARDWARE,
		nullptr,
		0,
		nullptr,
		0,
		D3D11_SDK_VERSION,
		&sd,
		&pSwap,
		&pDevice,
		nullptr,
		&pContext
	);
	// gain access to texture subresource in swap chain (back buffer)
    // SwapChain의 버퍼를 받아서 랜더타겟에 넘겨준다.
	ID3D11Resource* pBackBuffer = nullptr;
	pSwap->GetBuffer( 0,__uuidof(ID3D11Resource),reinterpret_cast<void**>(&pBackBuffer) );
	pDevice->CreateRenderTargetView(
		pBackBuffer,
		nullptr,
		&pTarget
	);
	pBackBuffer->Release();
}

Graphics::~Graphics()
{
	if( pTarget != nullptr )
	{
		pTarget->Release();
	}
	if( pContext != nullptr )
	{
		pContext->Release();
	}
	if( pSwap != nullptr )
	{
		pSwap->Release();
	}
	if( pDevice != nullptr )
	{
		pDevice->Release();
	}
}

void Graphics::EndFrame()
{
	pSwap->Present( 1u,0u );
}

void Graphics::ClearBuffer( float red,float green,float blue ) noexcept
{
	const float color[] = { red,green,blue,1.0f };
	pContext->ClearRenderTargetView( pTarget,color );
}
```

![](/assets/img/posts/directx/dxd-basic-12-1.gif){:class="img-fluid"}

이후에 포인터를 아래와 같이 관리하는데

```cpp
Microsoft::WRL::ComPtr<ID3D11Device> pDevice;
Microsoft::WRL::ComPtr<IDXGISwapChain> pSwap;
Microsoft::WRL::ComPtr<ID3D11DeviceContext> pContext;
Microsoft::WRL::ComPtr<ID3D11RenderTargetView> pTarget;
```

MS용 unique_ptr이라 생각하자