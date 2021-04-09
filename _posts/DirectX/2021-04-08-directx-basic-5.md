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

## 쉐이더에 데이터를 넣는 방법2

배열의 형태로 데이터를 넣을 수 있다.<br>
이런형태로 구현 예정

![](/assets/img/posts/directx/basic-5-2.png){:class="img-fluid"}

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
#include "pch.h"
#include "RootSignature.h"

void RootSignature::Init(ComPtr<ID3D12Device> device)
{
	CD3DX12_DESCRIPTOR_RANGE ranges[] =
	{
		CD3DX12_DESCRIPTOR_RANGE(D3D12_DESCRIPTOR_RANGE_TYPE_CBV, CBV_REGISTER_COUNT, 0),
		 // GPU레지스터 공간에 b0~b4 를 쓸 예정
	};

	// 이런식으로 GPU의 레지스터를 할당받는다고 알아두자.
	CD3DX12_ROOT_PARAMETER param[1];
	param[0].InitAsDescriptorTable(_countof(ranges), ranges);

	D3D12_ROOT_SIGNATURE_DESC sigDesc = CD3DX12_ROOT_SIGNATURE_DESC(_countof(param), param);
	sigDesc.Flags = D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT; // 입력 조립기 단계

	ComPtr<ID3DBlob> blobSignature;
	ComPtr<ID3DBlob> blobError;
	::D3D12SerializeRootSignature(&sigDesc, D3D_ROOT_SIGNATURE_VERSION_1, &blobSignature, &blobError);
	device->CreateRootSignature(0, blobSignature->GetBufferPointer(), blobSignature->GetBufferSize(), IID_PPV_ARGS(&_signature));
}
```

![](/assets/img/posts/directx/basic-5-3.png){:class="img-fluid"}

여기까지 끝남, 이제 Constant Buffer를 구현해보자.<br>
heap을 통해서 GPU에 복사하기 때문에 constant buffer 뿐만 아니라 desc. heap까지 만들어야 한다.<br>

![](/assets/img/posts/directx/basic-5-4.png){:class="img-fluid"}

```cpp
#pragma once

class ConstantBuffer
{
public:
	ConstantBuffer();
	~ConstantBuffer();

	void Init(uint32 size, uint32 count);

	void Clear();
	D3D12_CPU_DESCRIPTOR_HANDLE PushData(int32 rootParamIndex, void* buffer, uint32 size);

	D3D12_GPU_VIRTUAL_ADDRESS GetGpuVirtualAddress(uint32 index);
	D3D12_CPU_DESCRIPTOR_HANDLE GetCpuHandle(uint32 index);

private:
	void CreateBuffer();
	void CreateView();

private:
	ComPtr<ID3D12Resource>	_cbvBuffer;
	BYTE*					_mappedBuffer = nullptr;
	uint32					_elementSize = 0;
	uint32					_elementCount = 0;

	ComPtr<ID3D12DescriptorHeap>		_cbvHeap;
	D3D12_CPU_DESCRIPTOR_HANDLE			_cpuHandleBegin = {};
	// 버퍼의 시작점
	uint32								_handleIncrementSize = 0;
	// 버퍼 하나의 크기(버퍼 이동을 위해 갖고있음)

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
	CreateView();
}

void ConstantBuffer::CreateBuffer()
{
	uint32 bufferSize = _elementSize * _elementCount;
	// _elementSize : 버퍼하나의 사이즈
	// _elementCount : 버퍼를 몇개로 만들 것인가
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

void ConstantBuffer::Clear()
{
	_currentIndex = 0;
}

D3D12_CPU_DESCRIPTOR_HANDLE ConstantBuffer::PushData(int32 rootParamIndex, void* buffer, uint32 size)
{
	assert(_currentIndex < _elementSize);

	::memcpy(&_mappedBuffer[_currentIndex * _elementSize], buffer, size);

	D3D12_CPU_DESCRIPTOR_HANDLE cpuHandle = GetCpuHandle(_currentIndex);
	
	_currentIndex++;

	return cpuHandle;
}

D3D12_GPU_VIRTUAL_ADDRESS ConstantBuffer::GetGpuVirtualAddress(uint32 index)
{
	D3D12_GPU_VIRTUAL_ADDRESS objCBAddress = _cbvBuffer->GetGPUVirtualAddress();
	objCBAddress += index * _elementSize;
	return objCBAddress;
}

D3D12_CPU_DESCRIPTOR_HANDLE ConstantBuffer::GetCpuHandle(uint32 index)
{
	// desc. heap의 각 인덱스의 핸들을 리턴해준다
	return CD3DX12_CPU_DESCRIPTOR_HANDLE(_cpuHandleBegin, index * _handleIncrementSize);
}
```

![](/assets/img/posts/directx/basic-5-5.png){:class="img-fluid"}

TableDescriptorHeap만 구현하면 된다.

```cpp
#pragma once

class TableDescriptorHeap
{
public:
	void Init(uint32 count);

	void Clear();
	void SetCBV(D3D12_CPU_DESCRIPTOR_HANDLE srcHandle, CBV_REGISTER reg);
	void CommitTable();

	ComPtr<ID3D12DescriptorHeap> GetDescriptorHeap() { return _descHeap; }

	D3D12_CPU_DESCRIPTOR_HANDLE GetCPUHandle(CBV_REGISTER reg);

private:
	D3D12_CPU_DESCRIPTOR_HANDLE GetCPUHandle(uint32 reg);

private:

	ComPtr<ID3D12DescriptorHeap> _descHeap;
	uint64					_handleSize = 0;
	uint64					_groupSize = 0;
	uint64					_groupCount = 0;

	uint32					_currentGroupIndex = 0;
};
```

```cpp
#include "pch.h"
#include "TableDescriptorHeap.h"
#include "Engine.h"

void TableDescriptorHeap::Init(uint32 count)
{
	// 그룹을 몇개짜리로 만들까?
	_groupCount = count;

	D3D12_DESCRIPTOR_HEAP_DESC desc = {};
	desc.NumDescriptors = count * REGISTER_COUNT;
	desc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE;
	desc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV;

	DEVICE->CreateDescriptorHeap(&desc, IID_PPV_ARGS(&_descHeap));

	_handleSize = DEVICE->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV);
	_groupSize = _handleSize * REGISTER_COUNT;
}

void TableDescriptorHeap::Clear()
{
	_currentGroupIndex = 0;
}

void TableDescriptorHeap::SetCBV(D3D12_CPU_DESCRIPTOR_HANDLE srcHandle, CBV_REGISTER reg)
{
	D3D12_CPU_DESCRIPTOR_HANDLE destHandle = GetCPUHandle(reg);

	uint32 destRange = 1;
	uint32 srcRange = 1;

	// CPU자원을 GPU로 복사해 달라
	DEVICE->CopyDescriptors(1, &destHandle, &destRange, 1, &srcHandle, &srcRange, D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV);
}

void TableDescriptorHeap::CommitTable()
{
	// 레지스터로 올려보내는 작업
	D3D12_GPU_DESCRIPTOR_HANDLE handle = _descHeap->GetGPUDescriptorHandleForHeapStart();
	handle.ptr += _currentGroupIndex * _groupSize;
	CMD_LIST->SetGraphicsRootDescriptorTable(0, handle);

	_currentGroupIndex++;
}

D3D12_CPU_DESCRIPTOR_HANDLE TableDescriptorHeap::GetCPUHandle(CBV_REGISTER reg)
{
	return GetCPUHandle(static_cast<uint32>(reg));
}

D3D12_CPU_DESCRIPTOR_HANDLE TableDescriptorHeap::GetCPUHandle(uint32 reg)
{
	D3D12_CPU_DESCRIPTOR_HANDLE handle = _descHeap->GetCPUDescriptorHandleForHeapStart();
	handle.ptr += _currentGroupIndex * _groupSize;
	handle.ptr += reg * _handleSize;
	return handle;
}
```

```cpp
void CommandQueue::RenderBegin(const D3D12_VIEWPORT* vp, const D3D12_RECT* rect)
{
	// ...

	_cmdList->SetGraphicsRootSignature(ROOT_SIGNATURE.Get());
	GEngine->GetCB()->Clear();
	GEngine->GetTableDescHeap()->Clear();

	ID3D12DescriptorHeap* descHeap = GEngine->GetTableDescHeap()->GetDescriptorHeap().Get();
	_cmdList->SetDescriptorHeaps(1, &descHeap);
```

```cpp
void Mesh::Render()
{
	CMD_LIST->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
	CMD_LIST->IASetVertexBuffers(0, 1, &_vertexBufferView); // Slot: (0~15)

	// TODO
	// 1) Buffer에다가 데이터 세팅
	// 2) TableDescHeap에다가 CBV 전달
	// 3) 모두 세팅이 끝났으면 TableDescHeap 커밋
	{
		D3D12_CPU_DESCRIPTOR_HANDLE handle = GEngine->GetCB()->PushData(0, &_transform, sizeof(_transform));
		GEngine->GetTableDescHeap()->SetCBV(handle, CBV_REGISTER::b0);
	}
	{
		D3D12_CPU_DESCRIPTOR_HANDLE handle = GEngine->GetCB()->PushData(0, &_transform, sizeof(_transform));
		GEngine->GetTableDescHeap()->SetCBV(handle, CBV_REGISTER::b1);
	}

	GEngine->GetTableDescHeap()->CommitTable();

	CMD_LIST->DrawInstanced(_vertexCount, 1, 0, 0);
}
```

===

## 정리

게임의 시작(Init)부터 다시 보며 어디가 수정되었는지 확인.

게임이 초기화 될때 Mesh정보를 초기화 한다.(GPU에 Mesh정보를 올린다.)<br>
그리고 쉐이더 정보도 초기화 한다.

```cpp
void Game::Init(const WindowInfo& info)
{
	GEngine->Init(info);

	vector<Vertex> vec(3);
	vec[0].pos = Vec3(0.f, 0.5f, 0.5f);
	vec[0].color = Vec4(1.f, 0.f, 0.f, 1.f);
	vec[1].pos = Vec3(0.5f, -0.5f, 0.5f);
	vec[1].color = Vec4(0.f, 1.0f, 0.f, 1.f);
	vec[2].pos = Vec3(-0.5f, -0.5f, 0.5f);
	vec[2].color = Vec4(0.f, 0.f, 1.f, 1.f);
	mesh->Init(vec);

	shader->Init(L"..\\Resources\\Shader\\default.hlsli");

	GEngine->GetCmdQueue()->WaitSync();
}
```

이후 게임이 Update되는 과정에서 Mesh의 Offset정보를 Transfrom을 통해서 넘긴다.

```cpp
void Game::Update()
{
	GEngine->RenderBegin();

	shader->Update();

	{
		Transform t;
		t.offset = Vec4(0.75f, 0.f, 0.f, 0.f);
		mesh->SetTransform(t);

		mesh->Render();
	}

	{
		Transform t;
		t.offset = Vec4(0.f, 0.75f, 0.f, 0.f);
		mesh->SetTransform(t);

		mesh->Render();
	}

	GEngine->RenderEnd();
}
```

```cpp
void Mesh::Render()
{
	CMD_LIST->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
	CMD_LIST->IASetVertexBuffers(0, 1, &_vertexBufferView); // Slot: (0~15)

	// TODO
	// 1) Buffer에다가 데이터 세팅
	// 2) TableDescHeap에다가 CBV 전달
	// 3) 모두 세팅이 끝났으면 TableDescHeap 커밋
	{
		D3D12_CPU_DESCRIPTOR_HANDLE handle = GEngine->GetCB()->PushData(0, &_transform, sizeof(_transform));
		GEngine->GetTableDescHeap()->SetCBV(handle, CBV_REGISTER::b0);
	}
	{
		D3D12_CPU_DESCRIPTOR_HANDLE handle = GEngine->GetCB()->PushData(0, &_transform, sizeof(_transform));
		GEngine->GetTableDescHeap()->SetCBV(handle, CBV_REGISTER::b1);
	}

	GEngine->GetTableDescHeap()->CommitTable();

	CMD_LIST->DrawInstanced(_vertexCount, 1, 0, 0);
}
```

중요한 부분이 세 부분인데

1. PushData를 통해서 transform 정보를 넣는 부분
2. set Constant Buffer View를 통해 레지스터 값을 변경하는 부분
3. CommitTable을 호출하는 부분

```cpp
// 1. PushData를 통해서 transform 정보를 넣는 부분

D3D12_CPU_DESCRIPTOR_HANDLE ConstantBuffer::PushData(int32 rootParamIndex, void* buffer, uint32 size)
{
	assert(_currentIndex < _elementSize);

	// _mappedBuffer가 Constant Buffer View 이다.
	::memcpy(&_mappedBuffer[_currentIndex * _elementSize], buffer, size);

	// Constant Buffer View에 transform 데이터를 넣고

	// 해당 핸들정보를 리턴한다.
	D3D12_CPU_DESCRIPTOR_HANDLE cpuHandle = GetCpuHandle(_currentIndex);
	
	_currentIndex++;

	return cpuHandle;
}
```

```cpp
// 2. set Constant Buffer View를 통해 레지스터 값을 변경하는 부분

void TableDescriptorHeap::SetCBV(D3D12_CPU_DESCRIPTOR_HANDLE srcHandle, CBV_REGISTER reg)
{
	D3D12_CPU_DESCRIPTOR_HANDLE destHandle = GetCPUHandle(reg);

	uint32 destRange = 1;
	uint32 srcRange = 1;

	// CPU에서 관리하던 Constant Buffer View를 GPU로 옮겨달라
	DEVICE->CopyDescriptors(1, &destHandle, &destRange, 1, &srcHandle, &srcRange, D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV);
}
```

```cpp
// 3. CommitTable을 호출하는 부분

void TableDescriptorHeap::CommitTable()
{
	D3D12_GPU_DESCRIPTOR_HANDLE handle = _descHeap->GetGPUDescriptorHandleForHeapStart();
	handle.ptr += _currentGroupIndex * _groupSize;
	CMD_LIST->SetGraphicsRootDescriptorTable(0, handle);

	_currentGroupIndex++;
}
```