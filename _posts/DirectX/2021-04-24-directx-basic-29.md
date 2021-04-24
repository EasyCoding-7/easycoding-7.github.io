---
layout: post
title:  "(DirectX : Basic) 29. Orthographic Projection(직교투영)"
summary: ""
author: DirectX
date: '2021-04-24 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-29/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/19)

---

지금까지 사용하던 Projection(투영)은 원근투영, 멀리있으면 작게 가까이있으면 크게 나타내는 투영방법이다.<br>
그럼 직교투영은 거리에 상관없이 일정하게 나타나게하는 투영법이다. 예로들자면 게임화면의 UI등이 있다.

![](/assets/img/posts/directx/basic-29-1.png){:class="img-fluid"}

어떤 물체이든 깊이값은 상관이 없어진다는 특징이 있다.

아래와 같은 화면의 점 O를 출력한다고 가정하자

```

|-----------------------------|
|                             |
|               O(x, y)       |
|                             |
|                             | (h)
|                             |
|                             |
|-----------------------------|
              (w)
```

화면에 나타내려면 -1~1의 사이에 값을 대입해야한다.<br>
(x, y) 좌표를 -1~1사이의 좌표로 변환하려면 아래 식을 대입하면 된다.<br>
(f = far / n = near)

```
x = 2x/w
y = 2y/h
z = (z/(f-n))-(n/(f-n))
```

이 식을 행렬로 대입해보자.

```
2/w 0   0        0
0   2/h 0        0
0   0   1/(f-n)  0
0   0   -n/(f-n) 1
```

---

이번에는 두 가지 작업을 한다.

1. 카메라를 두 개로 나누고
2. 하나의 카메라는 UI(Orthographic Projection) 하나는 원근투영을 찍은 후 합친다.

## Code

우선은 UI인지 아닌지 판별을 위해 Layer를 만들자

```cpp
// SceneManager.h

enum
{
	MAX_LAYER = 32
};

class SceneManager
{
	// ...

	array<wstring, MAX_LAYER> _layerNames;
    // Layer를 관리
	map<wstring, uint8> _layerIndex;

    // ...
```

각 GameObject는 자신읜 LayerIndex를 갖고있어야 한다.

```cpp
class GameObject : public Object, public enable_shared_from_this<GameObject>
{
    // ...

	void SetLayerIndex(uint8 layer) { _layerIndex = layer; }
	uint8 GetLayerIndex() { return _layerIndex; }

    // ...

	bool _checkFrustum = true;
	uint8 _layerIndex = 0;
};
```

카메라의 경우 자신이 찍을 Object의 Layer를 알고있어야 한다.

```cpp
class Camera : public Component
{
public:
	// ...

	void SetCullingMaskLayerOnOff(uint8 layer, bool on)
	{
		if (on)
			_cullingMask |= (1 << layer);
		else
			_cullingMask &= ~(1 << layer);
	}
    // 특정 Layer만 찍을지 찍지말지 결정

	void SetCullingMaskAll() { SetCullingMask(UINT32_MAX); }
    // 아무것도 찍지 않겠다.
	void SetCullingMask(uint32 mask) { _cullingMask = mask; }
	bool IsCulled(uint8 layer) { return (_cullingMask & (1 << layer)) != 0; }
    // 찍을지 말지 물어본다.

private:
	PROJECTION_TYPE _type = PROJECTION_TYPE::PERSPECTIVE;

	// ...

	Frustum _frustum;
	uint32 _cullingMask = 0;

    // ...
```

```cpp
void Camera::Render()
{
	S_MatView = _matView;
	S_MatProjection = _matProjection;

	shared_ptr<Scene> scene = GET_SINGLE(SceneManager)->GetActiveScene();

	// TODO : Layer 구분
	const vector<shared_ptr<GameObject>>& gameObjects = scene->GetGameObjects();

	for (auto& gameObject : gameObjects)
	{
		if (gameObject->GetMeshRenderer() == nullptr)
			continue;

        // 찍을 대상인지 아닌지 확인
		if (IsCulled(gameObject->GetLayerIndex()))
			continue;

		if (gameObject->GetCheckFrustum())
		{
			if (_frustum.ContainsSphere(
				gameObject->GetTransform()->GetWorldPosition(),
				gameObject->GetTransform()->GetBoundingSphereRadius()) == false)
			{
				continue;
			}
		}

		gameObject->GetMeshRenderer()->Render();
	}
}
```

```cpp
void Camera::FinalUpdate()
{
	_matView = GetTransform()->GetLocalToWorldMatrix().Invert();

	float width = static_cast<float>(GEngine->GetWindow().width);
	float height = static_cast<float>(GEngine->GetWindow().height);

	if (_type == PROJECTION_TYPE::PERSPECTIVE)
		_matProjection = ::XMMatrixPerspectiveFovLH(_fov, width / height, _near, _far);
	else
        // 직교투영 처리방법
		_matProjection = ::XMMatrixOrthographicLH(width * _scale, height * _scale, _near, _far);

	_frustum.FinalUpdate();
}
```

이제 Layer를 만들어보자

```cpp
shared_ptr<Scene> SceneManager::LoadTestScene()
{
#pragma region LayerMask
	SetLayerName(0, L"Default");
	SetLayerName(1, L"UI");
#pragma endregion

    // ...

    // 카메라 선언
#pragma region Camera
	{
		shared_ptr<GameObject> camera = make_shared<GameObject>();
		camera->SetName(L"Main_Camera");
		camera->AddComponent(make_shared<Transform>());
		camera->AddComponent(make_shared<Camera>()); // Near=1, Far=1000, FOV=45도
		camera->AddComponent(make_shared<TestCameraScript>());
		camera->GetTransform()->SetLocalPosition(Vec3(0.f, 0.f, 0.f));
		uint8 layerIndex = GET_SINGLE(SceneManager)->LayerNameToIndex(L"UI");
		camera->GetCamera()->SetCullingMaskLayerOnOff(layerIndex, true); // UI는 안 찍음
		scene->AddGameObject(camera);
	}	
#pragma endregion

#pragma region UI_Camera
	{
		shared_ptr<GameObject> camera = make_shared<GameObject>();
		camera->SetName(L"Orthographic_Camera");
		camera->AddComponent(make_shared<Transform>());
		camera->AddComponent(make_shared<Camera>()); // Near=1, Far=1000, 800*600
		camera->GetTransform()->SetLocalPosition(Vec3(0.f, 0.f, 0.f));
		camera->GetCamera()->SetProjectionType(PROJECTION_TYPE::ORTHOGRAPHIC);
		uint8 layerIndex = GET_SINGLE(SceneManager)->LayerNameToIndex(L"UI");
		camera->GetCamera()->SetCullingMaskAll(); // 다 끄고
		camera->GetCamera()->SetCullingMaskLayerOnOff(layerIndex, false); // UI만 찍음
		scene->AddGameObject(camera);
	}
#pragma endregion

    // ...
```

UI가 될 Object추가

```cpp
#pragma region UI_Test
	{
		shared_ptr<GameObject> sphere = make_shared<GameObject>();
		sphere->SetLayerIndex(GET_SINGLE(SceneManager)->LayerNameToIndex(L"UI")); // UI
		sphere->AddComponent(make_shared<Transform>());
		sphere->GetTransform()->SetLocalScale(Vec3(100.f, 100.f, 100.f));
		sphere->GetTransform()->SetLocalPosition(Vec3(0, 0, 500.f));
		shared_ptr<MeshRenderer> meshRenderer = make_shared<MeshRenderer>();
		{
			shared_ptr<Mesh> mesh = GET_SINGLE(Resources)->LoadRectangleMesh();
			meshRenderer->SetMesh(mesh);
		}
		{
			shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"Forward");
			shared_ptr<Texture> texture = GET_SINGLE(Resources)->Load<Texture>(L"Leather", L"..\\Resources\\Texture\\Leather.jpg");
			shared_ptr<Material> material = make_shared<Material>();
			material->SetShader(shader);
			material->SetTexture(0, texture);
			meshRenderer->SetMaterial(material);
		}
		sphere->AddComponent(meshRenderer);
		scene->AddGameObject(sphere);
	}
#pragma endregion
```

Mesh도 추가

```cpp
shared_ptr<Mesh> Resources::LoadRectangleMesh()
{
	shared_ptr<Mesh> findMesh = Get<Mesh>(L"Rectangle");
	if (findMesh)
		return findMesh;

	float w2 = 0.5f;
	float h2 = 0.5f;

	vector<Vertex> vec(4);

	// 앞면
	vec[0] = Vertex(Vec3(-w2, -h2, 0), Vec2(0.0f, 1.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));
	vec[1] = Vertex(Vec3(-w2, +h2, 0), Vec2(0.0f, 0.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));
	vec[2] = Vertex(Vec3(+w2, +h2, 0), Vec2(1.0f, 0.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));
	vec[3] = Vertex(Vec3(+w2, -h2, 0), Vec2(1.0f, 1.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));

	vector<uint32> idx(6);

	// 앞면
	idx[0] = 0; idx[1] = 1; idx[2] = 2;
	idx[3] = 0; idx[4] = 2; idx[5] = 3;

	shared_ptr<Mesh> mesh = make_shared<Mesh>();
	mesh->Init(vec, idx);
	Add(L"Rectangle", mesh);

	return mesh;
}
```