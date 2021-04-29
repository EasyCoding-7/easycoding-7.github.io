---
layout: post
title:  "(Unreal : Basic) 22. Random Numbers"
summary: ""
author: Unreal
date: '2021-04-29 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-22/
---

```cpp
void AFloater::BeginPlay()
{
	Super::BeginPlay();
	
	float InitialX = FMath::FRand();
	float InitialY = FMath::FRand();
	float InitialZ = FMath::FRand();

	InitialLocation.X = InitialX;
	InitialLocation.Y = InitialY;
	InitialLocation.Z = InitialZ;
	InitialLocation *= 20.f;

    // ...
```

```cpp
void AFloater::BeginPlay()
{
	Super::BeginPlay();
	
	float InitialX = FMath::FRandRange(-500.f, 500.f);
	float InitialY = FMath::FRandRange(-500.f, 500.f);
	float InitialZ = FMath::FRandRange(-500.f, 500.f);

	InitialLocation.X = InitialX;
	InitialLocation.Y = InitialY;
	InitialLocation.Z = InitialZ;

    // ...
```
