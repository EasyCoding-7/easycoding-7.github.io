---
layout: post
title:  "(DirectX : Basic) 22. Lighting-2"
summary: ""
author: DirectX
date: '2021-04-15 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-22/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/15)

---

## Component에 Light를 생성

```cpp
#pragma once
#include "Component.h"

// 빛이 어떻게 올 것인가 정보
enum class LIGHT_TYPE : uint8
{
	DIRECTIONAL_LIGHT,
	POINT_LIGHT,
	SPOT_LIGHT,
};

// 빛마다 속성
struct LightColor
{
    // 여기서 드는 의문점 RGB라 Vec3이지 왜 Vec4인가?
    // bytes padding 때문임.
	Vec4	diffuse;
	Vec4	ambient;
	Vec4	specular;
};

// 쉐이더에 넘겨줄 빛의 정보
struct LightInfo
{
	LightColor	color;          // 빛의 속성
	Vec4		position;       // 빛의 위치   
	Vec4		direction;      // 빛의 방향
	int32		lightType;      // LIGHT_TYPE
	float		range;          // 빛의 최대 범위
	float		angle;          // 빛이 쏘는 각도
	int32		padding;        // 16 bytes 로 만들기 위한 padding
};

struct LightParams
{
	uint32		lightCount;     // 빛의 개수
	Vec3		padding;
	LightInfo	lights[50];     // 빛의 정보를 하나로 묶어서 보낸다
};

class Light : public Component
{
public:
	Light();
	virtual ~Light();

	virtual void FinalUpdate() override;

public:
	const LightInfo& GetLightInfo() { return _lightInfo; }

	void SetLightDirection(const Vec3& direction) { _lightInfo.direction = direction; }

	void SetDiffuse(const Vec3& diffuse) { _lightInfo.color.diffuse = diffuse; }
	void SetAmbient(const Vec3& ambient) { _lightInfo.color.ambient = ambient; }
	void SetSpecular(const Vec3& specular) { _lightInfo.color.specular = specular; }

	void SetLightType(LIGHT_TYPE type) { _lightInfo.lightType = static_cast<int32>(type); }
	void SetLightRange(float range) { _lightInfo.range = range; }
	void SetLightAngle(float angle) { _lightInfo.angle = angle; }

private:
	LightInfo _lightInfo = {};
};
```

```cpp
#include "pch.h"
#include "Light.h"
#include "Transform.h"

Light::Light() : Component(COMPONENT_TYPE::LIGHT)
{
}

Light::~Light()
{
}

void Light::FinalUpdate()
{
	_lightInfo.position = GetTransform()->GetWorldPosition();
}
```

---

## .hlsli 파일 수정

매번 default.hlsli에 데이터를 다 넣었는데 그러다 보면 default.hlsli의 코드양이 너무커지게 된다.<Br>
좀 나누는 과정이 필요

그리고 현재는 GPU 레지스터에 데이터를 보낼때 Descriptor Table만을 통해서 보내는데<br>
그러다 보니 갱신되지 않아도 되는 데이터 마저 Descriptor Table안에 딸려들어가며 매번 갱신해야하는 불편한 상황이 발생한다.<br>
갱신이 필요없는 부분은 Constant Buffer에 넣고 갱신이 지속적으로 일어나는 부분만 Descriptor Table로 보내게 하겠다.

```cpp
void RootSignature::CreateRootSignature()
{
    // Descriptor Table 선언
	CD3DX12_DESCRIPTOR_RANGE ranges[] =
	{
		CD3DX12_DESCRIPTOR_RANGE(D3D12_DESCRIPTOR_RANGE_TYPE_CBV, CBV_REGISTER_COUNT - 1, 1), // b1~b4
		CD3DX12_DESCRIPTOR_RANGE(D3D12_DESCRIPTOR_RANGE_TYPE_SRV, SRV_REGISTER_COUNT, 0), // t0~t4
	};

	CD3DX12_ROOT_PARAMETER param[2];

    // Constant Buffer 선언
	param[0].InitAsConstantBufferView(static_cast<uint32>(CBV_REGISTER::b0)); // b0
	param[1].InitAsDescriptorTable(_countof(ranges), ranges);	

	D3D12_ROOT_SIGNATURE_DESC sigDesc = CD3DX12_ROOT_SIGNATURE_DESC(_countof(param), param, 1, &_samplerDesc);
	sigDesc.Flags = D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT; // 입력 조립기 단계

	ComPtr<ID3DBlob> blobSignature;
	ComPtr<ID3DBlob> blobError;
	::D3D12SerializeRootSignature(&sigDesc, D3D_ROOT_SIGNATURE_VERSION_1, &blobSignature, &blobError);
	DEVICE->CreateRootSignature(0, blobSignature->GetBufferPointer(), blobSignature->GetBufferSize(), IID_PPV_ARGS(&_signature));
}
```

```cpp
void TableDescriptorHeap::Init(uint32 count)
{
	_groupCount = count;

	D3D12_DESCRIPTOR_HEAP_DESC desc = {};
	desc.NumDescriptors = count * (REGISTER_COUNT - 1); // b0는 전역
	desc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE;
	desc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV;

	DEVICE->CreateDescriptorHeap(&desc, IID_PPV_ARGS(&_descHeap));

	_handleSize = DEVICE->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV);
	_groupSize = _handleSize * (REGISTER_COUNT - 1); // b0는 전역
}
```

```cpp
D3D12_CPU_DESCRIPTOR_HANDLE TableDescriptorHeap::GetCPUHandle(uint8 reg)
{
	assert(reg > 0);    // b0를 선택하지 못하게 걸러야한다.
	D3D12_CPU_DESCRIPTOR_HANDLE handle = _descHeap->GetCPUDescriptorHandleForHeapStart();
	handle.ptr += _currentGroupIndex * _groupSize;
	handle.ptr += (reg - 1) * _handleSize;
	return handle;
}
```

```cpp
void Engine::Init(const WindowInfo& info)
{
	// ...

    // Light는 Constant Buffer로 선언
	CreateConstantBuffer(CBV_REGISTER::b0, sizeof(LightParams), 1);
	CreateConstantBuffer(CBV_REGISTER::b1, sizeof(TransformParams), 256);
	CreateConstantBuffer(CBV_REGISTER::b2, sizeof(MaterialParams), 256);
```

```cpp
// b0용 Constant Buffer 데이터 복사
void ConstantBuffer::SetGlobalData(void* buffer, uint32 size)
{
	assert(_elementSize == ((size + 255) & ~255));
	::memcpy(&_mappedBuffer[0], buffer, size);
	CMD_LIST->SetGraphicsRootConstantBufferView(0, GetGpuVirtualAddress(0));
}
```

```cpp
void Scene::PushLightData()
{
	LightParams lightParams = {};

	for (auto& gameObject : _gameObjects)
	{
		if (gameObject->GetLight() == nullptr)
			continue;

		const LightInfo& lightInfo = gameObject->GetLight()->GetLightInfo();

		lightParams.lights[lightParams.lightCount] = lightInfo;
		lightParams.lightCount++;
	}

	CONST_BUFFER(CONSTANT_BUFFER_TYPE::GLOBAL)->SetGlobalData(&lightParams, sizeof(lightParams));
}
```

현제는 쉐이더에 좌표계가 World, View, Projection이 완료된 matWVP가 들어가는데<br>
빛과 관련된 쉐이더 처리 혹은 이후의 다른 처리를 위해 World, View, Projection각각의 좌표계가 필요할지 모른다.<br>
각각의 좌표계를 쉐이더로 넘겨보자

우선 넘길 구조체를 생성

```cpp
// EnginePch.h

struct TransformParams
{
	Matrix matWorld;
	Matrix matView;
	Matrix matProjection;
	Matrix matWV;
	Matrix matWVP;
};
```

```cpp
void Transform::PushData()
{
	TransformParams transformParams = {};
	transformParams.matWorld = _matWorld;
	transformParams.matView = Camera::S_MatView;
	transformParams.matProjection = Camera::S_MatProjection;
	transformParams.matWV = _matWorld * Camera::S_MatView;
	transformParams.matWVP = _matWorld * Camera::S_MatView * Camera::S_MatProjection;

	CONST_BUFFER(CONSTANT_BUFFER_TYPE::TRANSFORM)->PushData(&transformParams, sizeof(transformParams));
}
```

```
// default.hlsli

struct VS_IN
{
    float3 pos : POSITION;
    float2 uv : TEXCOORD;
    float3 normal : NORMAL; // normal 벡터
};

struct VS_OUT
{
    float4 pos : SV_Position;
    float2 uv : TEXCOORD;
    float3 viewPos : POSITION;  // 빛이 딱 맞은 곳의 위치
    float3 viewNormal : NORMAL; // 빛이 딱 맞은 곳의 노멀 벡터
};

// ...
```

```

 (빛)(L)
 \              (정반사)(R)
  \      (N)      /
   \      |      /
    \     |     /
     \    |    /
      \(a)|   /
       \  |  /
--------------------------------------
 (   빛이 맞은 곳   )
```

```
// params.hlsli

cbuffer TRANSFORM_PARAMS : register(b1)
{
    // 각 좌표계를 넘긴다
    row_major matrix g_matWorld;
    row_major matrix g_matView;
    row_major matrix g_matProjection;
    row_major matrix g_matWV;
    row_major matrix g_matWVP;
};

// ...
```

## Packing Rules

HLSL에서는 16Bytes 바운더리로 정렬이 된다.<br>
무슨말일까?

```
cbuffer IE
{
    float4 Val1;
    float2 Val2;
    float2 Val3;
};
// IE는 2 * 16 bytes를 사용하게 된다.
// 이건 당연
/*
[ Val1(16bytes)               ]
[ Val2(8bytes) + Val3(8bytes) ]
*/

cbuffer IE2
{
    float2 Val1; // + padding
    float4 Val2;
    float2 Val3; // + padding
};
// IE는 3 * 16 bytes를 사용하게 된다.
/*
[ Val1(8bytes)  ]
[ Val2(16bytes) ]
[ Val3(8bytes)  ]
*/
```