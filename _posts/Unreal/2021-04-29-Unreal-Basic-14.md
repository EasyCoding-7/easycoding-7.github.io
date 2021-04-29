---
layout: post
title:  "(Unreal : Basic) 14 : F-Vectors 1"
summary: ""
author: Unreal
date: '2021-04-29 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-14/
---

## C++에서 Vector반영하기

```cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "Floater.h"

// Sets default values
AFloater::AFloater()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	StaticMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("CustomStaticMesh"));
}

// Called when the game starts or when spawned
void AFloater::BeginPlay()
{
	Super::BeginPlay();
	
	SetActorLocation(FVector(0.0f, 0.0f, 0.0f));
}

// Called every frame
void AFloater::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}
```

모양이 보이기 위해서 Static Mesh를 IcoSphere로 지정해 줘야함을 잊지말자.

![](/assets/img/posts/Unreal/basic-4-3-1.PNG){:class="img-fluid"}

이렇게 변수로 선언해도 됨.

```cpp
void AFloater::BeginPlay()
{
	Super::BeginPlay();
	
	FVector InitialLocation = FVector(0.0f, 0.0f, 0.0f);

	SetActorLocation(InitialLocation);
}
```