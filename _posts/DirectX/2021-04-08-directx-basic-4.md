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

* [참고사이트](https://ssinyoung.tistory.com/41)

쉐이더 단계에서 사용할 상수를 넘기는 법을 배운다.

예를들어 Vertext 쉐이딩 단계에서 상수를 넣고싶다.<br>
이럴경우 어떻게 상수를 넣어야할까?? 매개변수로 상수를 넣을 수 있을까?(물론없음)

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

---

## ConstantBuffer

우선 지난 장에서 쉐이딩 파이프라인을 RootSignature에서 관리하고<Br>
RootSignature를 통해서 쉐이딩 파이프라인 중 상수를 넣을 수 있다.

```cpp
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
void CommandQueue::RenderBegin(const D3D12_VIEWPORT* vp, const D3D12_RECT* rect) 
{ 
	// ... 
	// RootSignature가 사용되게 해달라고 전달 
	_cmdList->SetGraphicsRootSignature(ROOT_SIGNATURE.Get());
```

여기까지하면 RootSignature를 통해서 상수를 넘길준비까지 되었다.

이제 직접 상수를 넣는 부분이 필요한데

1. 데이터를 담을 GPU 메모리를 할당한다.
2. CPU의 데이터를 GPU 메모리에 데이터를 담는다
3. GPU 레지스터가 GPU 메모리에서 읽어와 쉐이딩 단계에서 사용한다.

### 데이터를 담을 GPU 메모리 할당

```cpp
// ConstantBuffer라는 클래스에서 담당
void ConstantBuffer::Init(uint32 size, uint32 count) 
{ 
	// 상수 버퍼는 256 바이트 배수로 만들어야 한다 
	// 0 256 512 768 
	_elementSize = (size + 255) & ~255; 
	_elementCount = count; 
	CreateBuffer(); 
}
```

```cpp
void ConstantBuffer::CreateBuffer() 
{ 
	uint32 bufferSize = _elementSize * _elementCount; 
	D3D12_HEAP_PROPERTIES heapProperty = CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD); 
	D3D12_RESOURCE_DESC desc = CD3DX12_RESOURCE_DESC::Buffer(bufferSize);


   // 메모리를 할당해 주세요 
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
```

### GPU 메모리에 데이터를 담는다
### GPU 레지스터가 GPU 메모리에서 읽어와 쉐이딩 단계에서 사용한다.

```cpp
void Mesh::Render() 
{ 
	// ... 

	// 1) Buffer에다가 데이터 세팅 
	// 2) Buffer의 주소를 register에다가 전송 
	GEngine->GetCB()->PushData(0, &_transform, sizeof(_transform)); 
	GEngine->GetCB()->PushData(1, &_transform, sizeof(_transform)); 

        // ...
}
```

```cpp
void ConstantBuffer::PushData(int32 rootParamIndex, void* buffer, uint32 size) 
{ 
	assert(_currentIndex < _elementSize);

	// 2. GPU 메모리에 데이터를 담는다
	::memcpy(&_mappedBuffer[_currentIndex * _elementSize], buffer, size); 
	D3D12_GPU_VIRTUAL_ADDRESS address = GetGpuVirtualAddress(_currentIndex);

	// 3. GPU 레지스터가 GPU 메모리에서 읽어와 쉐이딩 단계에서 사용한다. 
	CMD_LIST->SetGraphicsRootConstantBufferView(rootParamIndex, address); 
	_currentIndex++; 
}
```

---

## 주의할 점

구현중 이해가 되지 않는 부분이 있을 것이다.<br>
메모리 공간할당의 부분인데 사용될 메모리보다 GPU공간을 더 많이 할당한다.<br>
이런 할당을 하는 이유는 CPU->GPU 메모리 복사는 즉시 일어나고<br>
레지스터가 GPU메모리를 읽어가는 부분은 큐에 의해 동작하기에 즉시 동작하지 않는다

```cpp
void ConstantBuffer::PushData(int32 rootParamIndex, void* buffer, uint32 size) 
{ 
	assert(_currentIndex < _elementSize);
	// 메모리의 복사는 즉시 일어나는 부분이고 
	::memcpy(&_mappedBuffer[_currentIndex * _elementSize], buffer, size); 
	D3D12_GPU_VIRTUAL_ADDRESS address = GetGpuVirtualAddress(_currentIndex);


	// 레지스터에 주소값을 전달하는건 queue에 의해 나중에 일어나는 부분
	CMD_LIST->SetGraphicsRootConstantBufferView(rootParamIndex, address); 
	_currentIndex++; 
}
```

그럼 복사는 일어났는데 queue에 명령이 쌓여 내가 원하는 시점의 데이터를 못 가져오는 경우가 시차때문에 발생할수 있지 않을까?<br>
버퍼를 여러개만들어서 이 문제를 해결해야한다.

```cpp
void ConstantBuffer::CreateBuffer()  
{  
	uint32 bufferSize = _elementSize * _elementCount; 
	// 버퍼사이즈를 사용하는 레지스터 count만큼 잡음 
	D3D12_HEAP_PROPERTIES heapProperty = CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD);  
	D3D12_RESOURCE_DESC desc = CD3DX12_RESOURCE_DESC::Buffer(bufferSize);
```
