---
layout: post
title:  "(DirectX : Basic) 2. 장치 초기화"
summary: ""
author: DirectX
date: '2021-04-08 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-2/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/2)

---

DirectX를 사용하기 위해서 여러 클래스를 정의한다.<br>
총 5개의 클래스

1. Engine
2. Device
3. CommandQueue
4. SwapChain
5. DesciptorHeap

을 정의할 예정이고 모든 클래스를 관리하는 Engine클래스를 먼저 설명하겠다.

---

## Engine

```cpp
#pragma once

class Engine
{
public:

	void Init(const WindowInfo& info);
	void Render();

public:
	void RenderBegin();
	void RenderEnd();

	void ResizeWindow(int32 width, int32 height);

private:
	WindowInfo		_window;
	// 위도우 정보(크기, 핸들 등을 담는다)

	D3D12_VIEWPORT	_viewport = {};
	// x, y, width, height 를 정의
	D3D12_RECT		_scissorRect = {};
	// 역시 x, y, width, height 를 정의
	// 단, 랜더링을 어디서 부터할지를 scissorRect에서 정의한다.

	// 아래 네 가지 클래스로 DirectX를 컨트롤하게 되며
	// 네 클래스를 관리하는 것이 Engine 클래스 이다.
	shared_ptr<class Device> _device;
	shared_ptr<class CommandQueue> _cmdQueue;
	shared_ptr<class SwapChain> _swapChain;
	shared_ptr<class DescriptorHeap> _descHeap;
};
```

```cpp
#include "pch.h"
#include "Engine.h"
#include "Device.h"
#include "CommandQueue.h"
#include "SwapChain.h"
#include "DescriptorHeap.h"

void Engine::Init(const WindowInfo& info)
{
	_window = info;
	ResizeWindow(info.width, info.height);

	// 그려질 화면 크기를 설정
	_viewport = { 0, 0, static_cast<FLOAT>(info.width), static_cast<FLOAT>(info.height), 0.0f, 1.0f };
	_scissorRect = CD3DX12_RECT(0, 0, info.width, info.height);

	// 모든 클래스를 여기서 초기화 하며 필요한 정보를 넘긴다.
	_device = make_shared<Device>();
	_cmdQueue = make_shared<CommandQueue>();
	_swapChain = make_shared<SwapChain>();
	_descHeap = make_shared<DescriptorHeap>();

	_device->Init();
	_cmdQueue->Init(_device->GetDevice(), _swapChain, _descHeap);
	_swapChain->Init(info, _device->GetDXGI(), _cmdQueue->GetCmdQueue());
	_descHeap->Init(_device->GetDevice(), _swapChain);
}
```

```cpp
void Engine::RenderBegin()
{
	// width, height 정보를 queue로 넘긴다.
	_cmdQueue->RenderBegin(&_viewport, &_scissorRect);
}
```

queue를 먼저 잠깐보자면

```cpp
void CommandQueue::RenderBegin(const D3D12_VIEWPORT* vp, const D3D12_RECT* rect) 
{
	// ... 
	// 명령리스트(일단 받아들이자)에 Viewport, Scissor Rect를 넘겨서 어디까지 그려달라 명령한다.
	_cmdList->RSSetViewports(1, vp); 
	_cmdList->RSSetScissorRects(1, rect);
```

Viewport, Scissor Rect가 궁금하다면 아래를 참조하자.

* [참조](https://ssinyoung.tistory.com/39)

대략적 설명은.

* Viewport : 렌더링을 할 렌더타겟(후면버퍼) 영역을 나타낸다.
* ScissorRect : 렌더링에서 제거하지 않을 영역을 설정한다. ScissorRect에 포함되지 않는 영역은 렌더링(레스터라이저)에서 제거된다.

---

## Device

```cpp
#pragma once

// 인력 사무소
class Device
{
public:
	void Init();

	ComPtr<IDXGIFactory> GetDXGI() { return _dxgi; }
	ComPtr<ID3D12Device> GetDevice() { return _device; }

private:
	// - ComPtr 일종의 스마트 포인터라 이해하면 된다.

	ComPtr<ID3D12Debug>			_debugController;
	// 디버그 출력을 담당
	ComPtr<IDXGIFactory>		_dxgi; 
	// 화면 관련 기능들, DirectX 공통기능 API라 생각
	ComPtr<ID3D12Device>		_device; 
	// 각종 객체 생성, GPU와 통신을 담당하는 녀석
};
```

init 함수는 위 변수를 생성하는 부분이라 쉬워서 생략한다.

---

## CommandQueue

DirectX는 GPU에 실시간으로 명령을 보내는 것이 아니라<br>
Queue를 통해서 GPU에게 명령을 요청해 두고 GPU는 queue에서 명령을 꺼내서 처리하는 형식이다.<br>

어떻게 명령을 보내는지 간략하게 설명하자면

1. Alloc을 통해서 명령을 보낼 메모리 공간을 GPU에 할당한다
2. 명령을 list에 담는다
3. queue를 통해서 명령을 보낸다.

```cpp
class CommandQueue
{
	// ...

private:
	// CommandQueue : DX12에 등장
	// 외주를 요청할 때, 하나씩 요청하면 비효율적
	// [외주 목록]에 일감을 차곡차곡 기록했다가 한 방에 요청하는 것
	ComPtr<ID3D12CommandQueue>			_cmdQueue;
	// 일감을 보내는 녀석
	ComPtr<ID3D12CommandAllocator>		_cmdAlloc;
	// 일감이 쌓이는 메모리공간
	ComPtr<ID3D12GraphicsCommandList>	_cmdList;
	// 일감 리스트 목록

	// Fence : 울타리(?)
	// CPU / GPU 동기화를 위한 간단한 도구
	ComPtr<ID3D12Fence>					_fence;
	uint32								_fenceValue = 0;
	// 일감에 숫자를 지정해 자신이 명령한 일감이 맞는지 확인용
	HANDLE								_fenceEvent = INVALID_HANDLE_VALUE;

	// ...
};
```

1. Alloc을 통해서 명령을 보낼 메모리 공간을 GPU에 할당한다

```cpp
void CommandQueue::Init(ComPtr<ID3D12Device> device, shared_ptr<SwapChain> swapChain, shared_ptr<DescriptorHeap> descHeap) 
{ 
	// ... 

	// - D3D12_COMMAND_LIST_TYPE_DIRECT : GPU가 직접 실행하는 명령 목록 
	// 일감 메모리공간을 할당해 달라 
	device->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&_cmdAlloc)); 

	// list는 할당된 alloc에 담겠다. 알림( _cmdAlloc.Get() )
	device->CreateCommandList(0, D3D12_COMMAND_LIST_TYPE_DIRECT, _cmdAlloc.Get(), nullptr, IID_PPV_ARGS(&_cmdList));

	// ... 
}
```

```cpp
void CommandQueue::RenderBegin(const D3D12_VIEWPORT* vp, const D3D12_RECT* rect) 
{
	// 다시 명령을 보낼경우 alloc된 메모리와 list를 초기화 해줘야하며,
	// list에게는 다시 alloc된 메모리 공간정보를 알려줘야한다.
	_cmdAlloc->Reset(); 
	_cmdList->Reset(_cmdAlloc.Get(), nullptr);
	
	// ...
```

2. 명령을 list에 담는다

```cpp
void CommandQueue::RenderBegin(const D3D12_VIEWPORT* vp, const D3D12_RECT* rect) 
{
	// 명령 메모리 공간할당
	_cmdAlloc->Reset(); 
	_cmdList->Reset(_cmdAlloc.Get(), nullptr); 

	// barrier(장벽)을 두고 특정메모리 공간을 어떻게 쓸것인가를 알린다.
	D3D12_RESOURCE_BARRIER barrier = CD3DX12_RESOURCE_BARRIER::Transition( 
		_swapChain->GetCurrentBackBufferResource().Get(), 
		D3D12_RESOURCE_STATE_PRESENT, 		 // (before) 현재 출력중인 화면 
		D3D12_RESOURCE_STATE_RENDER_TARGET); // (after) GPU가 작업중인 공간 
	// 메모리 공간 _swapChain->GetCurrentBackBufferResource().Get()을
	// 현재 D3D12_RESOURCE_STATE_PRESENT로 쓰고 있는데
	// D3D12_RESOURCE_STATE_RENDER_TARGET로 쓰게 해달라

	// 위 화면교체 명령을 바로 실행하는 것이 아니라 List에 추가 해 놓는 형태이다. 
	_cmdList->ResourceBarrier(1, &barrier);
	// 여기가 핵심 명령리스트에 포함시킨다.

	// Set the viewport and scissor rect.  This needs to be reset whenever the command list is reset. 
	// 그려질 공간을 재 할당했다고 생각하자(여기에 대한 자세한 설명은 없음) 
	_cmdList->RSSetViewports(1, vp); 
	_cmdList->RSSetScissorRects(1, rect);

	// Specify the buffers we are going to render to. 
	D3D12_CPU_DESCRIPTOR_HANDLE backBufferView = _descHeap->GetBackBufferView(); 
	_cmdList->ClearRenderTargetView(backBufferView, Colors::LightSteelBlue, 0, nullptr); 
	_cmdList->OMSetRenderTargets(1, &backBufferView, FALSE, nullptr);
	// OMSetRenderTargets : Render-Target과 Depth-Stencil-View의 Merger (Output-Merger(OM))
	// Render-Target과 Depth-Stencil-View를 합쳐주는 부분인데 일단은 받아 들인다.

	// 어쨋든 중요한건 _cmdList에 명령을 담고 있는것을 주목해서 보자.
```

3. queue를 통해서 명령을 보낸다.

```cpp
void CommandQueue::RenderEnd() 
{ 
	D3D12_RESOURCE_BARRIER barrier = CD3DX12_RESOURCE_BARRIER::Transition( 
		_swapChain->GetCurrentBackBufferResource().Get(), 
		D3D12_RESOURCE_STATE_RENDER_TARGET, 	// (before) GPU가 작업중인 공간 
		D3D12_RESOURCE_STATE_PRESENT); 			// (after) 화면 출력중인 화면 


	// List에 명령을 넣는다. 
	_cmdList->ResourceBarrier(1, &barrier); 
    _cmdList->Close();
	// 이제 더이상 보낼 명령이 없다고 알림 

	// 커맨드 리스트 수행 
	ID3D12CommandList* cmdListArr[] = { _cmdList.Get() }; 
	_cmdQueue->ExecuteCommandLists(_countof(cmdListArr), cmdListArr);
	// 실행해 주세요 
	// 일감을 보낸다.

	_swapChain->Present(); 
	// Wait until frame commands are complete.  This waiting is inefficient and is 
	// done for simplicity.  Later we will show how to organize our rendering code 
	// so we do not have to wait per frame. 
	WaitSync(); 

	_swapChain->SwapIndex(); 
}
```

---

## SwapChain

스왑체인은 더블버퍼링을 담당하는 부분이다. 더 자세한 설명은 다른 블로그 참조...

```cpp
#include "pch.h"
#include "SwapChain.h"

void SwapChain::Init(const WindowInfo& info, ComPtr<IDXGIFactory> dxgi, ComPtr<ID3D12CommandQueue> cmdQueue)
{
	// 이전에 만든 정보 날린다
	_swapChain.Reset();

	DXGI_SWAP_CHAIN_DESC sd;
	sd.BufferDesc.Width = static_cast<uint32>(info.width); // 버퍼의 해상도 너비
	sd.BufferDesc.Height = static_cast<uint32>(info.height); // 버퍼의 해상도 높이
	sd.BufferDesc.RefreshRate.Numerator = 60; // 화면 갱신 비율
	sd.BufferDesc.RefreshRate.Denominator = 1; // 화면 갱신 비율
	sd.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM; // 버퍼의 디스플레이 형식
	sd.BufferDesc.ScanlineOrdering = DXGI_MODE_SCANLINE_ORDER_UNSPECIFIED;
	sd.BufferDesc.Scaling = DXGI_MODE_SCALING_UNSPECIFIED;
	sd.SampleDesc.Count = 1; // 멀티 샘플링 OFF
	sd.SampleDesc.Quality = 0;
	sd.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT; // 후면 버퍼에 렌더링할 것 
	sd.BufferCount = SWAP_CHAIN_BUFFER_COUNT; // 전면+후면 버퍼
	sd.OutputWindow = info.hwnd;
	sd.Windowed = info.windowed;
	sd.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD; // 전면 후면 버퍼 교체 시 이전 프레임 정보 버림
	sd.Flags = DXGI_SWAP_CHAIN_FLAG_ALLOW_MODE_SWITCH;

	// swapchain은 dxgi를 통해서 만든다.
	dxgi->CreateSwapChain(cmdQueue.Get(), &sd, &_swapChain);

	// 스왑체인의 메모리 주소를 _renderTargets에 할당한다.
	for (int32 i = 0; i < SWAP_CHAIN_BUFFER_COUNT; i++)
		_swapChain->GetBuffer(i, IID_PPV_ARGS(&_renderTargets[i]));
}
```

```cpp
void SwapChain::Present()
{
	// Present the frame.
	// 화면에 프리젠트(출력)해 달라
	_swapChain->Present(0, 0);
	// init에서 받은 핸들에 그리게 된다.
}
```

---

## DescriptorHeap(Render-Target View)

Render-Target의 핸들이라 생각. Render-Target에 명령을 넣고 싶다면 여기를 만지면 되는데 현재는 사용되는 부분이 없기에 우선 설명은 생략한다.