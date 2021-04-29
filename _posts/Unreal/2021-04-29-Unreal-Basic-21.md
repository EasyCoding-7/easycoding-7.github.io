---
layout: post
title:  "(Unreal : Basic) 21. Force and Torque"
summary: ""
author: Unreal
date: '2021-04-29 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-21/
---

## 힘(Force)

```cpp
void AFloater::BeginPlay()
{
	Super::BeginPlay();
	
	PlacedLocation = GetActorLocation();

	if (bInitializerFloaterLocations)
	{
		SetActorLocation(InitialLocation);
	}


    // InitialForce에 어떻게 힘들 가할지 넘긴다
	StaticMesh->AddForce(InitialForce);
}
```

```cpp
UPROPERTY(EditInstanceOnly, BlueprintReadWrite, Category = "Floater Variables")
FVector InitialForce;
```

![](/assets/img/posts/Unreal/Basic-21-1.PNG){:class="img-fluid"}

---

## 토크(Torque)

```cpp
void AFloater::BeginPlay()
{
	// ...

	StaticMesh->AddTorqueInDegrees(InitialTorque);
}
```

```cpp
UPROPERTY(EditInstanceOnly, BlueprintReadWrite, Category = "Floater Variables")
FVector InitialTorque;
```

토크를 많이주면 스핀을 한다.