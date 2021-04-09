---
layout: post
title:  "(DirectX : Basic) 4. Constant Buffer"
summary: ""
author: DirectX
date: '2021-04-08 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-4/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/4)

---

기존에 출력되었던 삼각형을 조금 이동시켜 보자.

우선 가장 간단하게 생각해 볼 수 있는 방법이 쉐이더에서 output값을 변경해보면 어떨까??

```
// ...

VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT)0;

    output.pos = float4(input.pos, 1.f);
    output.pos += ???;		// 여기를 변경할 수 있다면??
    output.color = input.color;
    output.color += ???;

    return output;
}

// ...
```

쉐이더 내부에서 값을 변경하는 방법에 대한 챕터이다.

---

## Dx12 rendering pipeline

우선, 렌더링 파이프라인은 어떤식으로 동작할까?

![](/assets/img/posts/directx/basic-4-1.png){:class="img-fluid"}

오른쪽 검은 네모가 렌더링 파이프라인 과정이고, 위에서 부터 아래로 내려간다고 생각하자<br>

Vertex, Index Buuffer의 경우 초기에 특정한 형태로 넘길수 있고<br>
렌더링 파이프라인 중간중간 RootSignature을 통해 값을 넣을수 있다.<br>

그렇다는 말은 RootSignature로 우리가 원하는 값을 쉐이더로 넘겨줘야할 거 같은데 어떻게 할까?

```cpp
#include "pch.h"
#include "RootSignature.h"

void RootSignature::Init(ComPtr<ID3D12Device> device)
{
	// 우선은 두 개의 값을 넘긴다고 알려줘야한다.

	CD3DX12_ROOT_PARAMETER param[2];
	param[0].InitAsConstantBufferView(0); 
	// 0번 -> b0 -> CBV(Constant Buffer View), CBV는 레지스터이름이 b로 시작함
	param[1].InitAsConstantBufferView(1); // 1번 -> b1 -> CBV

	D3D12_ROOT_SIGNATURE_DESC sigDesc = CD3DX12_ROOT_SIGNATURE_DESC(2, param);
	sigDesc.Flags = D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT; // 입력 조립기 단계

	ComPtr<ID3DBlob> blobSignature;
	ComPtr<ID3DBlob> blobError;
	::D3D12SerializeRootSignature(&sigDesc, D3D_ROOT_SIGNATURE_VERSION_1, &blobSignature, &blobError);
	device->CreateRootSignature(0, blobSignature->GetBufferPointer(), blobSignature->GetBufferSize(), IID_PPV_ARGS(&_signature));
}
```

```cpp
void CommandQueue::RenderBegin(const D3D12_VIEWPORT* vp, const D3D12_RECT* rect)
{
	// ...

	// RootSignature가 사용되게 해달라고 전달
	_cmdList->SetGraphicsRootSignature(ROOT_SIGNATURE.Get());
```

아직 b0, b1 레지스터에 데이터를 넣는작업은 하지 않음, 어디서 할까??

```cpp
void Mesh::Render()
{
	CMD_LIST->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
	CMD_LIST->IASetVertexBuffers(0, 1, &_vertexBufferView); // Slot: (0~15)

	// 뭔가 여기서 렌더링 하면서 레지스터에 값을 줘야 그려지는 동시에 작업이 될꺼 같긴한데...

	CMD_LIST->DrawInstanced(_vertexCount, 1, 0, 0);
}
```

![](/assets/img/posts/directx/basic-4-3.png){:class="img-fluid"}

우선, 위 그림대로 CPU(왼쪽)에서 생성한 데이터를 GPU의 Buffer에 담고 다시 Buffer에서 b0, b1 레지스터로 담는 과정이 필요하다.<br>
좀 더 정확히는 b0, b1에는 Buffer의 주소가 담기고 그 주소를 보고 GPU는 데이터를 받아와 연산을 하게 된다.

그럼 Buffer를 GPU에 할당하는게 먼저겠군??

```cpp
void Engine::Init(const WindowInfo& info)
{
	// ...
	_cb = make_shared<ConstantBuffer>();

	// ...
	_cb->Init(sizeof(Transform), 256);
}
```

```cpp
void Mesh::Render()
{
	CMD_LIST->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
	CMD_LIST->IASetVertexBuffers(0, 1, &_vertexBufferView); // Slot: (0~15)

	// TODO
	// 1) Buffer에다가 데이터 세팅
	// 2) Buffer의 주소를 register에다가 전송
	GEngine->GetCB()->PushData(0, &_transform, sizeof(_transform));
	GEngine->GetCB()->PushData(1, &_transform, sizeof(_transform));

	CMD_LIST->DrawInstanced(_vertexCount, 1, 0, 0);
}
```

```cpp
#pragma once

class ConstantBuffer
{
public:
	ConstantBuffer();
	~ConstantBuffer();

	void Init(uint32 size, uint32 count);

	void Clear();
	void PushData(int32 rootParamIndex, void* buffer, uint32 size);

	D3D12_GPU_VIRTUAL_ADDRESS GetGpuVirtualAddress(uint32 index);

private:
	void CreateBuffer();

private:
	ComPtr<ID3D12Resource>	_cbvBuffer;
	BYTE*					_mappedBuffer = nullptr;
	uint32					_elementSize = 0;
	uint32					_elementCount = 0;

	uint32					_currentIndex = 0;
};
```

```cpp
#include "pch.h"
#include "ConstantBuffer.h"
#include "Engine.h"

ConstantBuffer::ConstantBuffer()
{
}

ConstantBuffer::~ConstantBuffer()
{
	if (_cbvBuffer)
	{
		if (_cbvBuffer != nullptr)
			_cbvBuffer->Unmap(0, nullptr);

		_cbvBuffer = nullptr;
	}
}

void ConstantBuffer::Init(uint32 size, uint32 count)
{
	// 상수 버퍼는 256 바이트 배수로 만들어야 한다
	// 0 256 512 768
	_elementSize = (size + 255) & ~255;
	_elementCount = count;

	CreateBuffer();
}

void ConstantBuffer::CreateBuffer()
{
	uint32 bufferSize = _elementSize * _elementCount;
	D3D12_HEAP_PROPERTIES heapProperty = CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD);
	D3D12_RESOURCE_DESC desc = CD3DX12_RESOURCE_DESC::Buffer(bufferSize);

	DEVICE->CreateCommittedResource(
		&heapProperty,
		D3D12_HEAP_FLAG_NONE,
		&desc,
		D3D12_RESOURCE_STATE_GENERIC_READ,
		nullptr,
		IID_PPV_ARGS(&_cbvBuffer));

	_cbvBuffer->Map(0, nullptr, reinterpret_cast<void**>(&_mappedBuffer));
	// We do not need to unmap until we are done with the resource.  However, we must not write to
	// the resource while it is in use by the GPU (so we must use synchronization techniques).
}

void ConstantBuffer::Clear()
{
	_currentIndex = 0;
}

void ConstantBuffer::PushData(int32 rootParamIndex, void* buffer, uint32 size)
{
	assert(_currentIndex < _elementSize);

	::memcpy(&_mappedBuffer[_currentIndex * _elementSize], buffer, size);

	D3D12_GPU_VIRTUAL_ADDRESS address = GetGpuVirtualAddress(_currentIndex);
	CMD_LIST->SetGraphicsRootConstantBufferView(rootParamIndex, address);
	_currentIndex++;
}

D3D12_GPU_VIRTUAL_ADDRESS ConstantBuffer::GetGpuVirtualAddress(uint32 index)
{
	D3D12_GPU_VIRTUAL_ADDRESS objCBAddress = _cbvBuffer->GetGPUVirtualAddress();
	objCBAddress += index * _elementSize;
	return objCBAddress;
}
```

![](/assets/img/posts/directx/basic-4-2.png){:class="img-fluid"}