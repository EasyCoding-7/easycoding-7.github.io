---
layout: post
title:  "(Unreal : Basic) 20. Local Vs World"
summary: ""
author: Unreal
date: '2021-04-29 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-20/
---

Local, World 좌표계에 대한 설명 쉽기에 내용은 없음.

## Offset을 줘서 시작지점을 옮겨보자.

```cpp
void AFloater::BeginPlay()
{
	// ...

	FHitResult HitResult;
	FVector LocalOffset = FVector(200.f, 0.0f, 0.0f);
    
    // 로컬기준 이동
    //AddActorLocalOffset(LocalOffset, true, &HitResult);

    // 월드기준 이동
	AddActorWorldOffset(LocalOffset, true, &HitResult);
}
```

## 회전시키기

```cpp
void AFloater::BeginPlay()
{
	// ...

	FRotator Rotation = FRotator(30.f, 0.f, 0.f);
	//AddActorLocalRotation(Rotation);
	AddActorWorldRotation(Rotation);
}
```

좀 더 쉬운 확인을 위해 Floater에 XYZ축을 추가해 두었다.

![](/assets/img/posts/Unreal/Basic-20-1.PNG){:class="img-fluid"}