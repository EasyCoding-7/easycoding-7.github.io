---
layout: post
title:  "(DirectX : Basic) 26. Frustum Culling"
summary: ""
author: DirectX
date: '2021-04-15 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-26/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/18)

---

![](/assets/img/posts/directx/basic-26-1.png){:class="img-fluid"}

지금까지는 Mesh를 만들기만 하면 무조건 그리고 있다.<br>
지금부터 frustum culling을 통해 그릴놈 안그릴놈 구분해주는 기능이라 생각하면 되겠다.

---

일단 3차원 평면의 식을 아래와 같이 정의해보자.

`ax + by + cz + d = 0`

위 3차원 평면의 Normal Vector는 `N(a, b, c)`가 되며 `d`의 경우 원점에서 평면까지의 거리가 된다.<br>
증명을 해보자면 한 평면위의 두 점 `A(x1, y1, z1)`, `B(x2, y2, z2)`과 Normal Vector `N`을 내적시 0이 되면 `N(a, b, c)`가 Normal Vector임이 증명이 된다.<br>
`AB = (x2 - x1, y2 - y1, z2 - z1)`, `N(a, b, c)`를 내적해보자.<br>

```
AB * N = a(x2 - x1) + b(y2 - y1) + c(z2 - z1)
AB * N = (ax2+by2+cz2) - (ax1+by1+cz1)
// ax2+by2+cz2+d = 0
// ax1+by1+cz1+d = 0
// 이기에
// ax2+by2+cz2 = -d
// ax1+by1+cz1 = -d
AB * N = 0      // 증명 끝!
```

갑자기 이건 왜 한걸까??<br>
특정 평면을 기준으로 평면보다 멀리 혹은 가까이 있는지를 판별하기 위해서이다.

뭔소린가?? 좀 더 자세히 설명해 보자면

```
    \
     A1(x, y, z)
      \
     / \
  d /   \
   /     \
 O        \
           \
```

`OA1 * N` (벡터 `OA1`과 Normal Vector `N`의 내적)은 `d`(평면과 `O`과의 거리)가 된다<br>
그럼 평면위의 점이 아니라 좀 더 가까운 점 A2의 내적은?

```
        \
  A2(x, y z)
          \
           \
            \
             \
     O        \
               \
```

`ab + by + cz + d < 0`이 되게 된다.<br>
만약 평면보다 멀다면 `ab + by + cz + d > 0`

---

이제 카메라뷰의 각 평면(6개)의 Normal Vector를 구한 후 내적해서 0보다 큰지 작은지 구하면 된다.

![](/assets/img/posts/directx/basic-26-2.PNG){:class="img-fluid"}

그러면 그려야 할지 말지 결정을 위해 Projection -> View -> World로 변환이 필요하다

```cpp
#pragma once

enum PLANE_TYPE : uint8
{
	PLANE_FRONT,
	PLANE_BACK,
	PLANE_UP,
	PLANE_DOWN,
	PLANE_LEFT,
	PLANE_RIGHT,

	PLANE_END
};

class Frustum
{
public:
	void FinalUpdate();

    // 그려할 평면 내부에 있는지 확인
	bool ContainsSphere(const Vec3& pos, float radius);

private:
	array<Vec4, PLANE_END> _planes;
};
```

```cpp
#include "pch.h"
#include "Frustum.h"
#include "Camera.h"

void Frustum::FinalUpdate()
{
	Matrix matViewInv = Camera::S_MatView.Invert();
	Matrix matProjectionInv = Camera::S_MatProjection.Invert();
	Matrix matInv = matProjectionInv * matViewInv;

	vector<Vec3> worldPos =
	{
		::XMVector3TransformCoord(Vec3(-1.f, 1.f, 0.f), matInv),
		::XMVector3TransformCoord(Vec3(1.f, 1.f, 0.f), matInv),
		::XMVector3TransformCoord(Vec3(1.f, -1.f, 0.f), matInv),
		::XMVector3TransformCoord(Vec3(-1.f, -1.f, 0.f), matInv),
		::XMVector3TransformCoord(Vec3(-1.f, 1.f, 1.f), matInv),
		::XMVector3TransformCoord(Vec3(1.f, 1.f, 1.f), matInv),
		::XMVector3TransformCoord(Vec3(1.f, -1.f, 1.f), matInv),
		::XMVector3TransformCoord(Vec3(-1.f, -1.f, 1.f), matInv)
	};

    // 평면을 생성 : 점 세개를 넘긴다
	_planes[PLANE_FRONT] = ::XMPlaneFromPoints(worldPos[0], worldPos[1], worldPos[2]); // CW(시계방향)
	_planes[PLANE_BACK] = ::XMPlaneFromPoints(worldPos[4], worldPos[7], worldPos[5]); // CCW(반시계방향)
	_planes[PLANE_UP] = ::XMPlaneFromPoints(worldPos[4], worldPos[5], worldPos[1]); // CW
	_planes[PLANE_DOWN] = ::XMPlaneFromPoints(worldPos[7], worldPos[3], worldPos[6]); // CCW
	_planes[PLANE_LEFT] = ::XMPlaneFromPoints(worldPos[4], worldPos[0], worldPos[7]); // CW
	_planes[PLANE_RIGHT] = ::XMPlaneFromPoints(worldPos[5], worldPos[6], worldPos[1]); // CCW
}

bool Frustum::ContainsSphere(const Vec3& pos, float radius)
{
	for (const Vec4& plane : _planes)
	{
		// n = (a, b, c)
		Vec3 normal = Vec3(plane.x, plane.y, plane.z);

		// ax + by + cz + d > radius
		if (normal.Dot(pos) + plane.w > radius)
			return false;
	}

	return true;
}
```

```cpp
void Camera::Render()
{
	shared_ptr<Scene> scene = GET_SINGLE(SceneManager)->GetActiveScene();

	// TODO : Layer 구분
	const vector<shared_ptr<GameObject>>& gameObjects = scene->GetGameObjects();

	for (auto& gameObject : gameObjects)
	{
		if (gameObject->GetMeshRenderer() == nullptr)
			continue;

        // Frustum을 적용하지 말아야 할 object도 존재한다(skybox)
		if (gameObject->GetCheckFrustum())
		{
            // 랜더링 할지 말지 결정
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
