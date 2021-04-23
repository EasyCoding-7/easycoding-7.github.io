---
layout: post
title:  "(DirectX : Basic) 25. Skybox"
summary: ""
author: DirectX
date: '2021-04-15 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-25/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/17)

---

하늘을 처리해야 할텐데 어떻게 처리하나? 단순위 캐릭터 위에 그리면 될까??<br>
일단 캐릭터 머리위에 단순히 하늘을 그리는 형태로 만드는 것은 맞다<br>
쉬워 보이지만 몇가지 고려해야할 사항이 있는데

1. 캐릭터가 움직여서 하늘의 끝에 닿으면 안된다.
2. 하늘뒤에 어떠한 물체도 놓이게 해선 안된다(그럴필요가 없으니)

* 하늘을 처리하는 한 가지 팁은 하늘의 위치는 카메라의 기준과 동일(0, 0, 0) 우선 이걸 기억하자
* 따라서 Translation은 적용하지 않고 Rotation만 적용이 되게 만들면 된다.

## Culling

랜더링 과정에서 그릴지 말지를 결정

랜더링 파이프라인에서 아래와 같이 `CD3DX12_RASTERIZER_DESC`옵션을 넣는데 옵션의 의미는 아래와 같다

```cpp
void Shader::Init(const wstring& path, ShaderInfo info)
{
	// ..

	_pipelineDesc.RasterizerState = CD3DX12_RASTERIZER_DESC(D3D12_DEFAULT);

	// ...
```

```cpp
explicit CD3DX12_RASTERIZER_DESC( CD3DX12_DEFAULT )
{
	FillMode = D3D12_FILL_MODE_SOLID;
	CullMode = D3D12_CULL_MODE_BACK;
	FrontCounterClockwise = FALSE;
	// Front를 시계 방향(Clockwise)로 둔다
	// 반 시계 방향(CounterClockwise)는 Culling(랜더링에서 그리지 않겠다.)

	// ...
```

시계방향 반시계방향의 의미는 인덱스 버퍼에서 결정된다.

갑자기 culling은 왜 언급하나??<br>
하늘(skybox)는 카메라(캐릭터) 입장에서는 skybox안에서 그려줘야한다.<br>
따라서 culling이 되지 않게 구현하기 위해선 인덱스 방향을 반대로 줘야한다

```cpp
// 인덱스 버퍼 시계/반시계 방향에 따라 culling여부 판별
enum class RASTERIZER_TYPE
{
	CULL_NONE,
	CULL_FRONT,
	CULL_BACK,
	WIREFRAME,
};

// depth stencil 비교연산 처리방법 결정
// 같은 최대 깊이더라도 하늘을 마지막에 그려야함.
enum class DEPTH_STENCIL_TYPE
{
	LESS,
	LESS_EQUAL,
	GREATER,
	GREATER_EQUAL,
};
```

```cpp
void Shader::Init(const wstring& path, ShaderInfo info)
{
	//..

	switch (info.rasterizerType)
	{
	case RASTERIZER_TYPE::CULL_BACK:
		_pipelineDesc.RasterizerState.FillMode = D3D12_FILL_MODE_SOLID;
		_pipelineDesc.RasterizerState.CullMode = D3D12_CULL_MODE_BACK;
		break;
	case RASTERIZER_TYPE::CULL_FRONT:
		_pipelineDesc.RasterizerState.FillMode = D3D12_FILL_MODE_SOLID;
		_pipelineDesc.RasterizerState.CullMode = D3D12_CULL_MODE_FRONT;
		break;
	case RASTERIZER_TYPE::CULL_NONE:
		_pipelineDesc.RasterizerState.FillMode = D3D12_FILL_MODE_SOLID;
		_pipelineDesc.RasterizerState.CullMode = D3D12_CULL_MODE_NONE;
		break;
	case RASTERIZER_TYPE::WIREFRAME:
		_pipelineDesc.RasterizerState.FillMode = D3D12_FILL_MODE_WIREFRAME;
		_pipelineDesc.RasterizerState.CullMode = D3D12_CULL_MODE_NONE;
		break;
	}

	switch (info.depthStencilType)
	{
	case DEPTH_STENCIL_TYPE::LESS:
		_pipelineDesc.DepthStencilState.DepthEnable = TRUE;
		_pipelineDesc.DepthStencilState.DepthFunc = D3D12_COMPARISON_FUNC_LESS;
		break;
	case DEPTH_STENCIL_TYPE::LESS_EQUAL:
		_pipelineDesc.DepthStencilState.DepthEnable = TRUE;
		_pipelineDesc.DepthStencilState.DepthFunc = D3D12_COMPARISON_FUNC_LESS_EQUAL;
		break;
	case DEPTH_STENCIL_TYPE::GREATER:
		_pipelineDesc.DepthStencilState.DepthEnable = TRUE;
		_pipelineDesc.DepthStencilState.DepthFunc = D3D12_COMPARISON_FUNC_GREATER;
		break;
	case DEPTH_STENCIL_TYPE::GREATER_EQUAL:
		_pipelineDesc.DepthStencilState.DepthEnable = TRUE;
		_pipelineDesc.DepthStencilState.DepthFunc = D3D12_COMPARISON_FUNC_GREATER_EQUAL;
		break;
	}

	DEVICE->CreateGraphicsPipelineState(&_pipelineDesc, IID_PPV_ARGS(&_pipelineState));

    // ...
```

```cpp
shared_ptr<Scene> SceneManager::LoadTestScene()
{
	shared_ptr<Scene> scene = make_shared<Scene>();

#pragma region Camera
	shared_ptr<GameObject> camera = make_shared<GameObject>();
	camera->AddComponent(make_shared<Transform>());
	camera->AddComponent(make_shared<Camera>()); // Near=1, Far=1000, FOV=45도
	camera->AddComponent(make_shared<TestCameraScript>());
	camera->GetTransform()->SetLocalPosition(Vec3(0.f, 0.f, 0.f));
	scene->AddGameObject(camera);
#pragma endregion

#pragma region SkyBox
	{
		shared_ptr<GameObject> skybox = make_shared<GameObject>();
		skybox->AddComponent(make_shared<Transform>());
		shared_ptr<MeshRenderer> meshRenderer = make_shared<MeshRenderer>();
		{
			shared_ptr<Mesh> sphereMesh = GET_SINGLE(Resources)->LoadSphereMesh();
			meshRenderer->SetMesh(sphereMesh);
		}
		{
			shared_ptr<Shader> shader = make_shared<Shader>();
			shared_ptr<Texture> texture = make_shared<Texture>();
			shader->Init(L"..\\Resources\\Shader\\skybox.hlsli",
				{ RASTERIZER_TYPE::CULL_NONE, DEPTH_STENCIL_TYPE::LESS_EQUAL });
			// 쉐이더 init시 RASTERIZER_TYPE::CULL_NONE, DEPTH_STENCIL_TYPE::LESS_EQUAL 옵션을 넣는다.
			// RASTERIZER_TYPE::CULL_NONE : cull을 하지말라 (cull_front로 해도 동작함)
			texture->Init(L"..\\Resources\\Texture\\Sky01.jpg");
			shared_ptr<Material> material = make_shared<Material>();
			material->SetShader(shader);
			material->SetTexture(0, texture);
			meshRenderer->SetMaterial(material);
		}
		skybox->AddComponent(meshRenderer);
		scene->AddGameObject(skybox);
	}
#pragma endregion

    // ...
```

```
#ifndef _SKYBOX_HLSLI_
#define _SKYBOX_HLSLI_

#include "params.hlsli"

struct VS_IN
{
    float3 localPos : POSITION;
    float2 uv : TEXCOORD;
};

struct VS_OUT
{
    float4 pos : SV_Position;
    float2 uv : TEXCOORD;
};

VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT)0;

    // float4(input.localPos, 0) : Translation은 하지 않고 Rotation만 적용한다(마지막 값을 0으로 둠)
    float4 viewPos = mul(float4(input.localPos, 0), g_matView);
    float4 clipSpacePos = mul(viewPos, g_matProjection);

	// clipSpacePos.xyww : z(깊이)에 w를 넣는다
    // w/w=1이기 때문에 항상 깊이가 1로 유지된다
    output.pos = clipSpacePos.xyww;
    output.uv = input.uv;

    return output;
}

float4 PS_Main(VS_OUT input) : SV_Target
{
     float4 color = g_tex_0.Sample(g_sam_0, input.uv);
     return color;
}

#endif
```