---
layout: post
title:  "(DirectX : Basic) 3. 삼각형 그리기"
summary: ""
author: DirectX
date: '2021-04-08 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-3/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/3)

---

## DescriptorHeap과 SwapChain을 합쳐보자

생각해보면 둘의 기능이 유사하다.

* DescriptorHeap : RenderTargetView 관리
* SwapChain : RenderTarget 관리

나눠 관리할 필요가 없다.

```cpp
class SwapChain
{
public:
	void Init(const WindowInfo& info, ComPtr<ID3D12Device> device, ComPtr<IDXGIFactory> dxgi, ComPtr<ID3D12CommandQueue> cmdQueue);
	void Present();
	void SwapIndex();

	ComPtr<IDXGISwapChain> GetSwapChain() { return _swapChain; }
	ComPtr<ID3D12Resource> GetRenderTarget(int32 index) { return _rtvBuffer[index]; }

	ComPtr<ID3D12Resource> GetBackRTVBuffer() { return _rtvBuffer[_backBufferIndex]; }
	D3D12_CPU_DESCRIPTOR_HANDLE GetBackRTV() { return _rtvHandle[_backBufferIndex]; }

private:
	void CreateSwapChain(const WindowInfo& info, ComPtr<IDXGIFactory> dxgi, ComPtr<ID3D12CommandQueue> cmdQueue);
	void CreateRTV(ComPtr<ID3D12Device> device);

private:
	ComPtr<IDXGISwapChain>	_swapChain;
	
	ComPtr<ID3D12Resource>			_rtvBuffer[SWAP_CHAIN_BUFFER_COUNT];
	ComPtr<ID3D12DescriptorHeap>	_rtvHeap;
	D3D12_CPU_DESCRIPTOR_HANDLE		_rtvHandle[SWAP_CHAIN_BUFFER_COUNT];

	uint32					_backBufferIndex = 0;
};
```

```cpp
#include "pch.h"
#include "SwapChain.h"


void SwapChain::Init(const WindowInfo& info, ComPtr<ID3D12Device> device, ComPtr<IDXGIFactory> dxgi, ComPtr<ID3D12CommandQueue> cmdQueue)
{
	CreateSwapChain(info, dxgi, cmdQueue);
	CreateRTV(device);
}

void SwapChain::Present()
{
	// Present the frame.
	_swapChain->Present(0, 0);
}

void SwapChain::SwapIndex()
{
	_backBufferIndex = (_backBufferIndex + 1) % SWAP_CHAIN_BUFFER_COUNT;
}

void SwapChain::CreateSwapChain(const WindowInfo& info, ComPtr<IDXGIFactory> dxgi, ComPtr<ID3D12CommandQueue> cmdQueue)
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

	dxgi->CreateSwapChain(cmdQueue.Get(), &sd, &_swapChain);

	for (int32 i = 0; i < SWAP_CHAIN_BUFFER_COUNT; i++)
		_swapChain->GetBuffer(i, IID_PPV_ARGS(&_rtvBuffer[i]));
}

void SwapChain::CreateRTV(ComPtr<ID3D12Device> device)
{
	// Descriptor (DX12) = View (~DX11)
	// [서술자 힙]으로 RTV 생성
	// DX11의 RTV(RenderTargetView), DSV(DepthStencilView), 
	// CBV(ConstantBufferView), SRV(ShaderResourceView), UAV(UnorderedAccessView)를 전부!

	int32 rtvHeapSize = device->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_RTV);

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
		_rtvHandle[i] = CD3DX12_CPU_DESCRIPTOR_HANDLE(rtvHeapBegin, i * rtvHeapSize);
		device->CreateRenderTargetView(_rtvBuffer[i].Get(), nullptr, _rtvHandle[i]);
	}
}
```

---

## device, commandqueue, swapchain은 많이 사용되는데 좀더 쉽게 꺼내쓸수 있게 변경해보자.

```cpp
class Engine
{
public:

	void Init(const WindowInfo& info);
	void Render();

public:
	shared_ptr<Device> GetDevice() { return _device; }
	shared_ptr<CommandQueue> GetCmdQueue() { return _cmdQueue; }
	shared_ptr<SwapChain> GetSwapChain() { return _swapChain; }
	shared_ptr<RootSignature> GetRootSignature() { return _rootSignature; }
```

```cpp
// EnginePch.h

// ...
#define DEVICE			GEngine->GetDevice()->GetDevice()
#define CMD_LIST		GEngine->GetCmdQueue()->GetCmdList()
#define ROOT_SIGNATURE	GEngine->GetRootSignature()->GetSignature()
// ...
```

물론 이런스타일을 싫어하는 사람도 있음, 무조건 Init에서 받아서 shared_ptr로 클래스 내부에 저장해야 한다고 하는 사람도 있으나, 방식의 차이일 뿐...

---

## RootSignature

* [RootSignature에 대한 추가설명](https://ssinyoung.tistory.com/41)

루트 서명(RootSignature)은 리소스(데이터)들을 각 그래픽스 파이프라인 셰이더에 전달한다<br>
마치 매개변수를 함수에 전달하듯?(전달하는 방식은 많이 다르다)<br>
설명만 봐선 이해가 안되는데 이후에 사용하면서 추가적으로 설명하겠음.<br>

---

## Mesh 클래스

* Mesh : 일단은 3D정점의 집합이라 생각하자

```cpp
class Mesh 
{ 
public: 
	void Init(vector<Vertex>& vec); 
	void Render(); 
private: 
	ComPtr<ID3D12Resource>		_vertexBuffer; 
	// GPU에 할당할 Mesh 메모리(리소스)
	D3D12_VERTEX_BUFFER_VIEW	_vertexBufferView = {}; 
	// 할당된 메모리(리소스)의 핸들(View)
	uint32 _vertexCount = 0; 
};
```

```cpp
#include "pch.h"
#include "Mesh.h"
#include "Engine.h"

void Mesh::Init(vector<Vertex>& vec)
{
	_vertexCount = static_cast<uint32>(vec.size());
	uint32 bufferSize = _vertexCount * sizeof(Vertex);

    // GPU에 업로드용 메모리를 할당해 달라
	D3D12_HEAP_PROPERTIES heapProperty = CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD);
	D3D12_RESOURCE_DESC desc = CD3DX12_RESOURCE_DESC::Buffer(bufferSize);

	// GPU의 공간을 빌린다.
	DEVICE->CreateCommittedResource(
		&heapProperty,
		D3D12_HEAP_FLAG_NONE,
		&desc,
		D3D12_RESOURCE_STATE_GENERIC_READ,
		nullptr,
		IID_PPV_ARGS(&_vertexBuffer));  // _vertexBuffer : GPU공간의 메모리를 의미

	// Copy the triangle data to the vertex buffer.
    // _vertexBuffer는 GPU공간이라 그냥 복사가 불가능 아래의 과정이 필요하다
	void* vertexDataBuffer = nullptr;
	CD3DX12_RANGE readRange(0, 0); // We do not intend to read from this resource on the CPU.
	_vertexBuffer->Map(0, &readRange, &vertexDataBuffer);	// GPU공간에 접근할 권한을 잠깐 갖는다.
	::memcpy(vertexDataBuffer, &vec[0], bufferSize);		// 메모리 복사 후
	_vertexBuffer->Unmap(0, nullptr);						// 권한을 다시 돌려줌

	// Initialize the vertex buffer view.
	_vertexBufferView.BufferLocation = _vertexBuffer->GetGPUVirtualAddress();
	_vertexBufferView.StrideInBytes = sizeof(Vertex); // 정점 1개 크기
	_vertexBufferView.SizeInBytes = bufferSize; // 버퍼의 크기	
}

void Mesh::Render()
{
	// 커멘드 리스트로 그리기를 요청함을 주목하자
	CMD_LIST->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);  // Mesh가 어떤형태인지(기본은 삼각형(TRIANGLELIST))
	CMD_LIST->IASetVertexBuffers(0, 1, &_vertexBufferView); // Slot: (0~15)
	CMD_LIST->DrawInstanced(_vertexCount, 1, 0, 0); // 그려줭
}
```

---

## Shader

* 각 정점사이를 어떻게 그릴지에 대한 정보를 관리하는 클래스

```cpp
#pragma once

// [일감 기술서] 외주 인력(GPU)들이 뭘 해야할지 기술
class Shader
{
public:
	void Init(const wstring& path);     // 쉐이더 파일을 받아야 한다.
	void Update();

private:
	void CreateShader(const wstring& path, const string& name, const string& version, ComPtr<ID3DBlob>& blob, D3D12_SHADER_BYTECODE& shaderByteCode);
	void CreateVertexShader(const wstring& path, const string& name, const string& version);
	void CreatePixelShader(const wstring& path, const string& name, const string& version);

private:
	ComPtr<ID3DBlob>					_vsBlob;
	ComPtr<ID3DBlob>					_psBlob;
	ComPtr<ID3DBlob>					_errBlob;

	ComPtr<ID3D12PipelineState>			_pipelineState;
	D3D12_GRAPHICS_PIPELINE_STATE_DESC  _pipelineDesc = {};
};
```

```cpp
#include "pch.h"
#include "Shader.h"
#include "Engine.h"

void Shader::Init(const wstring& path)
{
	CreateVertexShader(path, "VS_Main", "vs_5_0");  
	// Vertex Shader 쉐이더파일 에서 VS_Main을 부른다.
	// vs_5_0 : GPU에게 Vertex를 어떻게 쉐이딩(그릴지) 알린다.
	CreatePixelShader(path, "PS_Main", "ps_5_0");   
	// Pixel Shader 쉐이더파일 에서 PS_Main을 부른다.
	// ps_5_0 : GPU에게 Pixel(Vertex사이의)을 어떻게 쉐이딩(그릴지) 알린다.


	D3D12_INPUT_ELEMENT_DESC desc[] =
	{
		{ "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
		{ "COLOR", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 12, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 }
	};

	_pipelineDesc.InputLayout = { desc, _countof(desc) };
	_pipelineDesc.pRootSignature = ROOT_SIGNATURE.Get();

	_pipelineDesc.RasterizerState = CD3DX12_RASTERIZER_DESC(D3D12_DEFAULT);
	_pipelineDesc.BlendState = CD3DX12_BLEND_DESC(D3D12_DEFAULT);
	_pipelineDesc.DepthStencilState.DepthEnable = FALSE;
	_pipelineDesc.DepthStencilState.StencilEnable = FALSE;
	_pipelineDesc.SampleMask = UINT_MAX;
	_pipelineDesc.PrimitiveTopologyType = D3D12_PRIMITIVE_TOPOLOGY_TYPE_TRIANGLE;
	_pipelineDesc.NumRenderTargets = 1;
	_pipelineDesc.RTVFormats[0] = DXGI_FORMAT_R8G8B8A8_UNORM;
	_pipelineDesc.SampleDesc.Count = 1;

	DEVICE->CreateGraphicsPipelineState(&_pipelineDesc, IID_PPV_ARGS(&_pipelineState));
}

void Shader::Update()
{
	CMD_LIST->SetPipelineState(_pipelineState.Get());
	// 쉐이더가 변경되어 파이프라인의 상태가 변경되었음을 알린다. 
}
```

쉐이더 파일

```
struct VS_IN
{
    float3 pos : POSITION;
    float4 color : COLOR;
};

struct VS_OUT
{
    float4 pos : SV_Position;
    float4 color : COLOR;
};

VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT)0;

    output.pos = float4(input.pos, 1.f);
    output.color = input.color;

	// 입력들어온데로 거의 그대로 넘긴다
    return output;
}

float4 PS_Main(VS_OUT input) : SV_Target
{
	// 역시 들어온데로 거의그대로 넘긴다
    return input.color;
}
```

---

## 실행 화면

![](/assets/img/posts/directx/basic-3-1.png){:class="img-fluid"}