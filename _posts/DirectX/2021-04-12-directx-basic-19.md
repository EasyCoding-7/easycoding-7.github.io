---
layout: post
title:  "(DirectX : Basic) 19. Camera"
summary: ""
author: DirectX
date: '2021-04-12 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-19/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/13)

---

## 수학과 관련된 라이브러리를 받아온다.

[DirectXTK12](https://github.com/microsoft/DirectXTK)에서 SimpleMath의 h/cpp/inl을 가져온다.

---

```cpp
// EnginePch.h

// ...

using Vec2		= DirectX::SimpleMath::Vector2;
using Vec3		= DirectX::SimpleMath::Vector3;
using Vec4		= DirectX::SimpleMath::Vector4;
using Matrix	= DirectX::SimpleMath::Matrix;

// ...
```

참고 union을 쓰는 이유?

```cpp
union
{
    struct
    {
        float _11, _12, _13, _14;
        float _21, _22, _23, _24;
        float _31, _32, _33, _34;
        float _41, _42, _43, _44;
    };
    float m[4][4];
};
```

변수는 하나만 생성되고 접근을 두 가지방법으로 가능<br>
`_11`로 접근가능 `m[0][0]`으로도 접근가능

---

## Transform 클래스

```cpp
#pragma once
#include "Component.h"

class Transform : public Component
{
public:
	Transform();
	virtual ~Transform();

	virtual void FinalUpdate() override;
	void PushData();

public:
	// Parent 기준
	const Vec3& GetLocalPosition() { return _localPosition; }
	const Vec3& GetLocalRotation() { return _localRotation; }
	const Vec3& GetLocalScale() { return _localScale; }

	const Matrix& GetLocalToWorldMatrix() { return _matWorld; }

	// World에서 자신의 위치
	const Vec3& GetWorldPosition() { return _matWorld.Translation(); }

	// Right, Up, Look
	Vec3 GetRight() { return _matWorld.Right(); }
	Vec3 GetUp() { return _matWorld.Up(); }
	Vec3 GetLook() { return _matWorld.Backward(); }

	// SRT 정의
	void SetLocalPosition(const Vec3& position) { _localPosition = position; }
	void SetLocalRotation(const Vec3& rotation) { _localRotation = rotation; }
	void SetLocalScale(const Vec3& scale) { _localScale = scale; }

public:
	void SetParent(shared_ptr<Transform> parent) { _parent = parent; }
	weak_ptr<Transform> GetParent() { return _parent; }

private:
	// Parent 기준, Wrold기준 아님
	Vec3 _localPosition = {};
	Vec3 _localRotation = {};
	Vec3 _localScale = { 1.f, 1.f, 1.f };

	// Parent 기준 Matrix
	Matrix _matLocal= {};

	// World 기준 Matrix
	Matrix _matWorld = {};

	weak_ptr<Transform> _parent;
};
```

```cpp
#include "pch.h"
#include "Transform.h"
#include "Engine.h"
#include "Camera.h"

Transform::Transform() : Component(COMPONENT_TYPE::TRANSFORM)
{

}

Transform::~Transform()
{

}

void Transform::FinalUpdate()
{
	// (S) Scale
	Matrix matScale = Matrix::CreateScale(_localScale);

	// (R) Rotation
	Matrix matRotation = Matrix::CreateRotationX(_localRotation.x);
	matRotation *= Matrix::CreateRotationY(_localRotation.y);
	matRotation *= Matrix::CreateRotationZ(_localRotation.z);

	// (T) Translation
	Matrix matTranslation = Matrix::CreateTranslation(_localPosition);

	// Parent를 기준으로한 Local Matrix임을 잊지 말자
	_matLocal = matScale * matRotation * matTranslation;
	_matWorld = _matLocal;

	shared_ptr<Transform> parent = GetParent().lock();

	// Parent가 없다면 그냥 Local이 World Matrix가 된다
	if (parent != nullptr)
	{
		// World Matrix는 여기서 정의
		_matWorld *= parent->GetLocalToWorldMatrix();
	}
}

void Transform::PushData()
{
	// 결국, GPU에서 필요한건 어떻게 Projection되느냐 이기에 WVP Matrix를 보내준다.
	// 그리고 View, Projection은 Camera Component에서 관리한다.
	Matrix matWVP = _matWorld * Camera::S_MatView * Camera::S_MatProjection;
	CONST_BUFFER(CONSTANT_BUFFER_TYPE::TRANSFORM)->PushData(&matWVP, sizeof(matWVP));
}
```

```
cbuffer TRANSFORM_PARAMS : register(b0)
{
    row_major matrix matWVP;
    // row_major : DirectX 기준에서는 행을 기준으로 데이터를 읽고 m[0][0] -> m[0][1] -> m[0][2] ...
    // 쉐이더는 열을 기준으로 데이터를 읽는데 m[0][0] -> m[1][0] -> m[2][0] ...
    // 행을 기준으로 데이터를 읽어달라는 명령
};

cbuffer MATERIAL_PARAMS : register(b1)
{
    int int_0;
    int int_1;
    int int_2;
    int int_3;
    int int_4;
    float float_0;
    float float_1;
    float float_2;
    float float_3;
    float float_4;
};

Texture2D tex_0 : register(t0);
Texture2D tex_1 : register(t1);
Texture2D tex_2 : register(t2);
Texture2D tex_3 : register(t3);
Texture2D tex_4 : register(t4);

SamplerState sam_0 : register(s0);

struct VS_IN
{
    float3 pos : POSITION;
    float4 color : COLOR;
    float2 uv : TEXCOORD;
};

struct VS_OUT
{
    float4 pos : SV_Position;
    float4 color : COLOR;
    float2 uv : TEXCOORD;
};

VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT)0;

    output.pos = mul(float4(input.pos, 1.f), matWVP);
	// 좌표에 matMVP를 반영해 달라.
    output.color = input.color;
    output.uv = input.uv;

    return output;
}

float4 PS_Main(VS_OUT input) : SV_Target
{
    float4 color = tex_0.Sample(sam_0, input.uv);
    return color;
}
```

---

## Camera 클래스

```cpp
#pragma once
#include "Component.h"

enum class PROJECTION_TYPE
{
	PERSPECTIVE, // 원근 투영
	ORTHOGRAPHIC, // 직교 투영
};

class Camera : public Component
{
public:
	Camera();
	virtual ~Camera();

	virtual void FinalUpdate() override;
	void Render();

private:
	PROJECTION_TYPE _type = PROJECTION_TYPE::PERSPECTIVE;

	float _near = 1.f;
	float _far = 1000.f;
	float _fov = XM_PI / 4.f;
	float _scale = 1.f;

	Matrix _matView = {};
	Matrix _matProjection = {};

public:
	// TEMP
	static Matrix S_MatView;
	static Matrix S_MatProjection;
};
```

```cpp
#include "pch.h"
#include "Camera.h"
#include "Transform.h"
#include "Scene.h"
#include "SceneManager.h"
#include "GameObject.h"
#include "MeshRenderer.h"
#include "Engine.h"

Matrix Camera::S_MatView;
Matrix Camera::S_MatProjection;

Camera::Camera() : Component(COMPONENT_TYPE::CAMERA)
{
}

Camera::~Camera()
{
}

void Camera::FinalUpdate()
{
	// World Matrix를 역행렬취하면 View가 나온다.
	_matView = GetTransform()->GetLocalToWorldMatrix().Invert();

	float width = static_cast<float>(GEngine->GetWindow().width);
	float height = static_cast<float>(GEngine->GetWindow().height);

	// Projection 좌표계는 DirectX지원함수로 구한다.
	if (_type == PROJECTION_TYPE::PERSPECTIVE)
		_matProjection = ::XMMatrixPerspectiveFovLH(_fov, width / height, _near, _far);
	else
		// 직교투영, 원근법을 적용하지 않음.
		_matProjection = ::XMMatrixOrthographicLH(width * _scale, height * _scale, _near, _far);

	// 여기는 임시로 사용하는 부분이다.
	S_MatView = _matView;
	S_MatProjection = _matProjection;
}

void Camera::Render()
{
	shared_ptr<Scene> scene = GET_SINGLE(SceneManager)->GetActiveScene();

	// TODO : Layer 구분
	const vector<shared_ptr<GameObject>>& gameObjects = scene->GetGameObjects();

	for (auto& gameObject : gameObjects)
	{
		if (gameObject->GetMeshRenderer() == nullptr)
			continue;

		gameObject->GetMeshRenderer()->Render();
	}
}
```

앞에서 나왔지만 메모리 올리는 부분을 다시 보자

```cpp
void Transform::PushData()
{
	// WVP한 최종 결과물을 넘긴다.
	Matrix matWVP = _matWorld * Camera::S_MatView * Camera::S_MatProjection;
	CONST_BUFFER(CONSTANT_BUFFER_TYPE::TRANSFORM)->PushData(&matWVP, sizeof(matWVP));
}
```

```cpp
shared_ptr<Scene> SceneManager::LoadTestScene()
{
	// ...

	gameObject->AddComponent(make_shared<Transform>());
	shared_ptr<Transform> transform = gameObject->GetTransform();
	transform->SetLocalPosition(Vec3(0.f, 100.f, 200.f));
	transform->SetLocalScale(Vec3(100.f, 100.f, 1.f));

	// ...

#pragma region Camera
	shared_ptr<GameObject> camera = make_shared<GameObject>();
	camera->AddComponent(make_shared<Transform>());
	camera->AddComponent(make_shared<Camera>()); // Near=1, Far=1000, FOV=45도
	camera->AddComponent(make_shared<TestCameraScript>());
	camera->GetTransform()->SetLocalPosition(Vec3(0.f, 100.f, 0.f));
	scene->AddGameObject(camera);
#pragma endregion

	// ...
```

## 카메라를 움직이게 해보자.

```cpp
#pragma once
#include "MonoBehaviour.h"

class TestCameraScript : public MonoBehaviour
{
public:
	TestCameraScript();
	virtual ~TestCameraScript();

	virtual void LateUpdate() override;

private:
	float		_speed = 100.f;
};
```

```cpp
#include "pch.h"
#include "TestCameraScript.h"
#include "Transform.h"
#include "Camera.h"
#include "GameObject.h"
#include "Input.h"
#include "Timer.h"

TestCameraScript::TestCameraScript()
{
}

TestCameraScript::~TestCameraScript()
{
}

void TestCameraScript::LateUpdate()
{
	Vec3 pos = GetTransform()->GetLocalPosition();

	if (INPUT->GetButton(KEY_TYPE::W))
		pos += GetTransform()->GetLook() * _speed * DELTA_TIME;

	if (INPUT->GetButton(KEY_TYPE::S))
		pos -= GetTransform()->GetLook() * _speed * DELTA_TIME;

	if (INPUT->GetButton(KEY_TYPE::A))
		pos -= GetTransform()->GetRight() * _speed * DELTA_TIME;

	if (INPUT->GetButton(KEY_TYPE::D))
		pos += GetTransform()->GetRight() * _speed * DELTA_TIME;

	if (INPUT->GetButton(KEY_TYPE::Q))
	{
		Vec3 rotation = GetTransform()->GetLocalRotation();
		rotation.x += DELTA_TIME * 0.5f;
		GetTransform()->SetLocalRotation(rotation);
	}

	if (INPUT->GetButton(KEY_TYPE::E))
	{
		Vec3 rotation = GetTransform()->GetLocalRotation();
		rotation.x -= DELTA_TIME * 0.5f;
		GetTransform()->SetLocalRotation(rotation);
	}

	GetTransform()->SetLocalPosition(pos);
}
```