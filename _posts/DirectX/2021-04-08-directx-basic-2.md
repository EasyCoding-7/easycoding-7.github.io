---
layout: post
title:  "(DirectX : Basic) 2. 장치 초기화"
summary: ""
author: DirectX
date: '2021-04-08 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-2/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/2)

---

사실 이번장에서 배울내용은 초보자 입장에서 다 알 필요는 없다.<br>
우선 이런게 있다 정도만 알아두고 앞으로 진행해가며 몰랐던 부분을 채워나간다고 생각하자<br>

```cpp
// EnginePch.h

// ...

// 윈도우 상태를 넘길 struct 구성
struct WindowInfo
{
	HWND	hwnd; // 출력 윈도우
	int32	width; // 너비
	int32	height; // 높이
	bool	windowed; // 창모드 or 전체화면
};

extern unique_ptr<class Engine> GEngine;        // class Engine으로 선언한거는 전방선언, 현 시점에선 Engine을 모르기 때문.
```

```cpp
#include "pch.h"
#include "EnginePch.h"
#include "Engine.h"

// 전역으로 Engine을 선언
unique_ptr<Engine> GEngine = make_unique<Engine>();
```

```cpp
class Engine
{
public:

	void Init(const WindowInfo& info);      // Init에서 윈도우 정보를 저장
	void Render();

    // ...

private:
	// 그려질 화면 크기 관련
	WindowInfo		_window;

    // 아직은 잘 모르겠지만 D3D12_VIEWPORT, D3D12_RECT를 선언
    D3D12_VIEWPORT	_viewport = {};
	D3D12_RECT		_scissorRect = {};
```

```cpp
void Engine::Init(const WindowInfo& info)
{
	_window = info;
	ResizeWindow(info.width, info.height);

	// 그려질 화면 크기를 설정
	_viewport = { 0, 0, static_cast<FLOAT>(info.width), static_cast<FLOAT>(info.height), 0.0f, 1.0f };
	_scissorRect = CD3DX12_RECT(0, 0, info.width, info.height);

	// ...
```

```cpp
class Engine
{
	// ...

	// 아래 네 가지 클래스의 구현이 필요하다
	shared_ptr<class Device> _device;
	shared_ptr<class CommandQueue> _cmdQueue;
	shared_ptr<class SwapChain> _swapChain;
	shared_ptr<class DescriptorHeap> _descHeap;
};
```

```cpp
void Engine::Init(const WindowInfo& info)
{
	// ...

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

---

## Device

```cpp
#pragma once


// 인력 사무소 : GPU와 대화를 할 녀석
class Device
{
public:
	void Init();

	ComPtr<IDXGIFactory> GetDXGI() { return _dxgi; }
	ComPtr<ID3D12Device> GetDevice() { return _device; }

private:
	// COM(Component Object Model)
	// - DX의 프로그래밍 언어 독립성과 하위 호환성을 가능하게 하는 기술
	// - COM 객체(COM 인터페이스)를 사용. 세부사항은 우리한테 숨겨짐
	// - ComPtr 일종의 스마트 포인터
	ComPtr<ID3D12Debug>			_debugController;
	ComPtr<IDXGIFactory>		_dxgi; // 화면 관련 기능들
	ComPtr<ID3D12Device>		_device; // 각종 객체 생성
};
```

```cpp
void Device::Init()
{
	// D3D12 디버그층 활성화
	// - VC++ 출력창에 상세한 디버깅 메시지 출력
	// - riid : 디바이스의 COM ID
	// - ppDevice : 생성된 장치가 매개변수에 설정
#ifdef _DEBUG
	::D3D12GetDebugInterface(IID_PPV_ARGS(&_debugController));
	_debugController->EnableDebugLayer();
#endif

	// DXGI(DirectX Graphics Infrastructure)
	// Direct3D와 함께 쓰이는 API
	// - 전체 화면 모드 전환
	// - 지원되는 디스플레이 모드 열거 등
	// CreateDXGIFactory
	// - riid : 디바이스의 COM ID
	// - ppDevice : 생성된 장치가 매개변수에 설정
	::CreateDXGIFactory(IID_PPV_ARGS(&_dxgi));

	// device에서 모든 기능을 처리할수 없기에 _dxgi도 생성


	// CreateDevice
	// - 디스플레이 어댑터(그래픽 카드)를 나타내는 객체
	// - pAdapter : nullptr 지정하면 시스템 기본 디스플레이 어댑터
	// - MinimumFeatureLevel : 응용 프로그램이 요구하는 최소 기능 수준 (구닥다리 걸러낸다)
	// - riid : 디바이스의 COM ID
	// - ppDevice : 생성된 장치가 매개변수에 설정
	::D3D12CreateDevice(nullptr, D3D_FEATURE_LEVEL_11_0, IID_PPV_ARGS(&_device));

	// 그래픽카드에 요청시 _device을 통해서 한다
}
```

---

## CommandQueue

```cpp
#pragma once

class SwapChain;
class DescriptorHeap;

class CommandQueue
{
public:
	~CommandQueue();

	void Init(ComPtr<ID3D12Device> device, shared_ptr<SwapChain> swapChain, shared_ptr<DescriptorHeap> descHeap);
	void WaitSync();

	void RenderBegin(const D3D12_VIEWPORT* vp, const D3D12_RECT* rect);
	void RenderEnd();

	ComPtr<ID3D12CommandQueue> GetCmdQueue() { return _cmdQueue; }

private:
	// CommandQueue : DX12에 등장
	// 외주를 요청할 때, 하나씩 요청하면 비효율적
	// [외주 목록]에 일감을 차곡차곡 기록했다가 한 방에 요청하는 것
	ComPtr<ID3D12CommandQueue>			_cmdQueue;		// 일감을 넣는곳
	ComPtr<ID3D12CommandAllocator>		_cmdAlloc;		// 일감 메모리공간할당
	ComPtr<ID3D12GraphicsCommandList>	_cmdList;		// 일감 리스트

	// Fence : 울타리(?)
	// CPU / GPU 동기화를 위한 간단한 도구
	ComPtr<ID3D12Fence>					_fence;			// 외주를 준 일감이 끝날때까지 기다리게 할 때
	uint32								_fenceValue = 0;
	HANDLE								_fenceEvent = INVALID_HANDLE_VALUE;

	shared_ptr<SwapChain>		_swapChain;
	shared_ptr<DescriptorHeap>	_descHeap;
};
```

```cpp
void CommandQueue::Init(ComPtr<ID3D12Device> device, shared_ptr<SwapChain> swapChain, shared_ptr<DescriptorHeap> descHeap)
{
	_swapChain = swapChain;
	_descHeap = descHeap;

	D3D12_COMMAND_QUEUE_DESC queueDesc = {};
	queueDesc.Type = D3D12_COMMAND_LIST_TYPE_DIRECT;
	queueDesc.Flags = D3D12_COMMAND_QUEUE_FLAG_NONE;

	// 일단은 device로 뭔가 작업중임을 기억하자

	device->CreateCommandQueue(&queueDesc, IID_PPV_ARGS(&_cmdQueue));

	// - D3D12_COMMAND_LIST_TYPE_DIRECT : GPU가 직접 실행하는 명령 목록
	device->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&_cmdAlloc));

	// GPU가 하나인 시스템에서는 0으로
	// DIRECT or BUNDLE
	// Allocator
	// 초기 상태 (그리기 명령은 nullptr 지정)
	device->CreateCommandList(0, D3D12_COMMAND_LIST_TYPE_DIRECT, _cmdAlloc.Get(), nullptr, IID_PPV_ARGS(&_cmdList));

	// CommandList는 Close / Open 상태가 있는데
	// Open 상태에서 Command를 넣다가 Close한 다음 제출하는 개념
	_cmdList->Close();

	// CreateFence
	// - CPU와 GPU의 동기화 수단으로 쓰인다
	device->CreateFence(0, D3D12_FENCE_FLAG_NONE, IID_PPV_ARGS(&_fence));
	_fenceEvent = ::CreateEvent(nullptr, FALSE, FALSE, nullptr);
}
```

```cpp
void CommandQueue::WaitSync()
{
	// Advance the fence value to mark commands up to this fence point.
	// Event의 번호를 매긴다
	_fenceValue++;

	// Add an instruction to the command queue to set a new fence point.  Because we 
	// are on the GPU timeline, the new fence point won't be set until the GPU finishes
	// processing all the commands prior to this Signal().
	_cmdQueue->Signal(_fence.Get(), _fenceValue);

	// Wait until the GPU has completed commands up to this fence point.
	if (_fence->GetCompletedValue() < _fenceValue)
	{
		// Fire event when GPU hits current fence.  
		_fence->SetEventOnCompletion(_fenceValue, _fenceEvent);

		// Wait until the GPU hits current fence event is fired.
		::WaitForSingleObject(_fenceEvent, INFINITE);
	}
}
```

---

## SwapChain

```cpp
#pragma once


// 교환 사슬
// [외주 과정]
// - 현재 게임 세상에 있는 상황을 묘사
// - 어떤 공식으로 어떻게 계산할지 던져줌
// - GPU가 열심히 계산 (외주)
// - 결과물 받아서 화면에 그려준다

// [외주 결과물]을 어디에 받지?
// - 어떤 종이(Buffer)에 그려서 건내달라고 부탁해보자
// - 특수 종이를 만들어서 -> 처음에 건내주고 -> 결과물을 해당 종이에 받는다 OK
// - 우리 화면에 특수 종이(외주 결과물) 출력해준다

// [?]
// - 그런데 화면에 현재 결과물 출력하는 와중에, 다음 화면도 외주를 맡겨야 함
// - 현재 화면 결과물은 이미 화면 출력에 사용중
// - 특수 종이를 2개 만들어서, 하나는 현재 화면을 그려주고, 하나는 외주 맡기고...
// - Double Buffering!

// - [0] [1]
// 현재 화면 [1]  <-> GPU 작업중 [1] BackBuffer

class SwapChain
{
public:
	void Init(const WindowInfo& info, ComPtr<IDXGIFactory> dxgi, ComPtr<ID3D12CommandQueue> cmdQueue);
	void Present();
	void SwapIndex();

	ComPtr<IDXGISwapChain> GetSwapChain() { return _swapChain; }
	ComPtr<ID3D12Resource> GetRenderTarget(int32 index) { return _renderTargets[index]; }

	uint32 GetCurrentBackBufferIndex() { return _backBufferIndex; }
	ComPtr<ID3D12Resource> GetCurrentBackBufferResource() { return _renderTargets[_backBufferIndex]; }

private:
	ComPtr<IDXGISwapChain>	_swapChain;
	ComPtr<ID3D12Resource>	_renderTargets[SWAP_CHAIN_BUFFER_COUNT];
	uint32					_backBufferIndex = 0;
};
```

```cpp
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


	// device가 아니라 dxgi를 이용해서 만들었음을 기억
	dxgi->CreateSwapChain(cmdQueue.Get(), &sd, &_swapChain);

	// swapChain에서 생성된 버퍼를 renderTarget에 저장
	for (int32 i = 0; i < SWAP_CHAIN_BUFFER_COUNT; i++)
		_swapChain->GetBuffer(i, IID_PPV_ARGS(&_renderTargets[i]));
}
```

---

## DescriptorHeap

```cpp
#pragma once


// [기안서]
// 외주를 맡길 때 이런 저런 정보들을 같이 넘겨줘야 하는데,
// 아무 형태로나 요청하면 못 알아먹는다
// - 각종 리소스를 어떤 용도로 사용하는지 꼼꼼하게 적어서 넘겨주는 용도
class DescriptorHeap // View와도 동일한 용어
{
public:
	void Init(ComPtr<ID3D12Device> device, shared_ptr<class SwapChain> swapChain);

	D3D12_CPU_DESCRIPTOR_HANDLE		GetRTV(int32 idx) { return _rtvHandle[idx]; }

	D3D12_CPU_DESCRIPTOR_HANDLE		GetBackBufferView();

private:
	ComPtr<ID3D12DescriptorHeap>	_rtvHeap;
	uint32							_rtvHeapSize = 0;
	D3D12_CPU_DESCRIPTOR_HANDLE		_rtvHandle[SWAP_CHAIN_BUFFER_COUNT];

	shared_ptr<class SwapChain>		_swapChain;
};
```

```cpp
void DescriptorHeap::Init(ComPtr<ID3D12Device> device, shared_ptr<SwapChain> swapChain)
{
	_swapChain = swapChain;

	// Descriptor (DX12) = View (~DX11)
	// [서술자 힙]으로 RTV 생성
	// DX11의 RTV(RenderTargetView), DSV(DepthStencilView), 
	// CBV(ConstantBufferView), SRV(ShaderResourceView), UAV(UnorderedAccessView)를 전부!

	_rtvHeapSize = device->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_RTV);

	D3D12_DESCRIPTOR_HEAP_DESC rtvDesc;
	rtvDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_RTV;
	rtvDesc.NumDescriptors = SWAP_CHAIN_BUFFER_COUNT;
	rtvDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;
	rtvDesc.NodeMask = 0;

	// 같은 종류의 데이터끼리 배열로 관리
	// RTV 목록 : [ ] [ ]
	device->CreateDescriptorHeap(&rtvDesc, IID_PPV_ARGS(&_rtvHeap));

	D3D12_CPU_DESCRIPTOR_HANDLE rtvHeapBegin = _rtvHeap->GetCPUDescriptorHandleForHeapStart();

	for (int i = 0; i < SWAP_CHAIN_BUFFER_COUNT; i++)
	{
		_rtvHandle[i] = CD3DX12_CPU_DESCRIPTOR_HANDLE(rtvHeapBegin, i * _rtvHeapSize);
		device->CreateRenderTargetView(swapChain->GetRenderTarget(i).Get(), nullptr, _rtvHandle[i]);
	}
}
```

---

## Render 구현

```cpp
class Engine
{
public:

	void Init(const WindowInfo& info);
	void Render();

public:
	void RenderBegin();
	void RenderEnd();

	// ...
```

```cpp
void Engine::Render()
{
	RenderBegin();

	// TODO : 나머지 물체들 그려준다

	RenderEnd();
}

void Engine::RenderBegin()
{
	_cmdQueue->RenderBegin(&_viewport, &_scissorRect);
}

void Engine::RenderEnd()
{
	_cmdQueue->RenderEnd();
}
```

```cpp
void CommandQueue::RenderBegin(const D3D12_VIEWPORT* vp, const D3D12_RECT* rect)
{
	_cmdAlloc->Reset();
	_cmdList->Reset(_cmdAlloc.Get(), nullptr);

	D3D12_RESOURCE_BARRIER barrier = CD3DX12_RESOURCE_BARRIER::Transition(
		_swapChain->GetCurrentBackBufferResource().Get(),
		D3D12_RESOURCE_STATE_PRESENT, 			// 화면 출력 - before
		D3D12_RESOURCE_STATE_RENDER_TARGET); 	// 외주 결과물 - after

	// 리소스를 밀어넣음.
	_cmdList->ResourceBarrier(1, &barrier);

	// Set the viewport and scissor rect.  This needs to be reset whenever the command list is reset.
	_cmdList->RSSetViewports(1, vp);
	_cmdList->RSSetScissorRects(1, rect);

	// Specify the buffers we are going to render to.
	D3D12_CPU_DESCRIPTOR_HANDLE backBufferView = _descHeap->GetBackBufferView();
	_cmdList->ClearRenderTargetView(backBufferView, Colors::LightSteelBlue, 0, nullptr);
	_cmdList->OMSetRenderTargets(1, &backBufferView, FALSE, nullptr);
}

void CommandQueue::RenderEnd()
{
	D3D12_RESOURCE_BARRIER barrier = CD3DX12_RESOURCE_BARRIER::Transition(
		_swapChain->GetCurrentBackBufferResource().Get(),
		D3D12_RESOURCE_STATE_RENDER_TARGET, // 외주 결과물
		D3D12_RESOURCE_STATE_PRESENT); 		// 화면 출력
		// 뒤에있던 애를 앞으로 당김

	_cmdList->ResourceBarrier(1, &barrier);
	_cmdList->Close();

	// 커맨드 리스트 수행
	ID3D12CommandList* cmdListArr[] = { _cmdList.Get() };
	_cmdQueue->ExecuteCommandLists(_countof(cmdListArr), cmdListArr);

	_swapChain->Present();

	// Wait until frame commands are complete.  This waiting is inefficient and is
	// done for simplicity.  Later we will show how to organize our rendering code
	// so we do not have to wait per frame.

	// 실행이 될때까지 기다린다.
	WaitSync();

	_swapChain->SwapIndex();
}
```