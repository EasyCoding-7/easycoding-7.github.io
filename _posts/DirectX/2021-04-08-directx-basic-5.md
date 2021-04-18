---
layout: post
title:  "(DirectX : Basic) 5. Root Signature"
summary: ""
author: DirectX
date: '2021-04-08 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-5/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/5)

---

이번에는 Constant Buffer를 배열의 형태로 사용해 보자.

```cpp
// EnginePch.h

// ...

enum class CBV_REGISTER
{
	// 데이터를 넣는데 사용할 레지스터
	b0,
	b1,
	b2,
	b3,
	b4,

	END
};

enum
{
	SWAP_CHAIN_BUFFER_COUNT = 2,
	CBV_REGISTER_COUNT = CBV_REGISTER::END,
	REGISTER_COUNT = CBV_REGISTER::END,
};

// ...
```

```cpp
void RootSignature::Init(ComPtr<ID3D12Device> device) 
{ 
	void RootSignature::Init(ComPtr<ID3D12Device> device)
{
	CD3DX12_DESCRIPTOR_RANGE ranges[] =
	{
		CD3DX12_DESCRIPTOR_RANGE(D3D12_DESCRIPTOR_RANGE_TYPE_CBV, CBV_REGISTER_COUNT, 0),
		 // GPU레지스터 공간에 b0~b4 를 쓸 예정
	};

	CD3DX12_ROOT_PARAMETER param[1];
	param[0].InitAsDescriptorTable(_countof(ranges), ranges);

	D3D12_ROOT_SIGNATURE_DESC sigDesc = CD3DX12_ROOT_SIGNATURE_DESC(_countof(param), param);
	sigDesc.Flags = D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT;

	// ...
}
```

기존 코드(Constant Buffer)와 차이점은 Constant Buffer View를 직접 param에 넣는지 혹은 DescriptorTable(View Table)형태로 넣는지의 차이다.<br>
Constant Buffer를 배열의 형태로 쓰기위해선 Descritor Table형태로 넣어야 한다.

```cpp
// Constant Buffer View로 넘기는 기존 형태
void RootSignature::Init(ComPtr<ID3D12Device> device) 
{ 
	// 우선은 두 개의 값을 넘긴다고 알려줘야한다. 
	CD3DX12_ROOT_PARAMETER param[2]; 
	param[0].InitAsConstantBufferView(0);  
	// 0번 -> b0 -> CBV(Constant Buffer View), CBV는 레지스터이름이 b로 시작함 
	param[1].InitAsConstantBufferView(1); // 1번 -> b1 -> CBV 
	D3D12_ROOT_SIGNATURE_DESC sigDesc = CD3DX12_ROOT_SIGNATURE_DESC(2, param);

	// ...
```

```cpp
// GPU에 메모리를 할당하는 방법은 기존과 동일하며
void ConstantBuffer::CreateBuffer()
{
	uint32 bufferSize = _elementSize * _elementCount;
	D3D12_HEAP_PROPERTIES heapProperty = CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD);
	D3D12_RESOURCE_DESC desc = CD3DX12_RESOURCE_DESC::Buffer(bufferSize);

	// 버퍼를 할당해 달라.
	DEVICE->CreateCommittedResource(
		&heapProperty,
		D3D12_HEAP_FLAG_NONE,
		&desc,
		D3D12_RESOURCE_STATE_GENERIC_READ,
		nullptr,
		IID_PPV_ARGS(&_cbvBuffer));

	// 이 버퍼를 GPU와 통신하는데 쓸 것이다 선언
	_cbvBuffer->Map(0, nullptr, reinterpret_cast<void**>(&_mappedBuffer));
	// We do not need to unmap until we are done with the resource.  However, we must not write to
	// the resource while it is in use by the GPU (so we must use synchronization techniques).
}
```

```cpp
// 이 View는 위에서 할당한 Table을 컨트롤하기 위한 View이다.

void ConstantBuffer::CreateView()
{
	D3D12_DESCRIPTOR_HEAP_DESC cbvDesc = {};
	cbvDesc.NumDescriptors = _elementCount;
	cbvDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;
	cbvDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV;
	DEVICE->CreateDescriptorHeap(&cbvDesc, IID_PPV_ARGS(&_cbvHeap));

	_cpuHandleBegin = _cbvHeap->GetCPUDescriptorHandleForHeapStart();
	_handleIncrementSize = DEVICE->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV);

	for (uint32 i = 0; i < _elementCount; ++i)
	{
		D3D12_CPU_DESCRIPTOR_HANDLE cbvHandle = GetCpuHandle(i);

		D3D12_CONSTANT_BUFFER_VIEW_DESC cbvDesc = {};
		cbvDesc.BufferLocation = _cbvBuffer->GetGPUVirtualAddress() + static_cast<uint64>(_elementSize) * i;
		cbvDesc.SizeInBytes = _elementSize;   // CB size is required to be 256-byte aligned.

		DEVICE->CreateConstantBufferView(&cbvDesc, cbvHandle);
	}
}
```