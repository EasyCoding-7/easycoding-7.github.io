---
layout: post
title:  "(Unreal : Basic) 17. Intro to Collision"
summary: ""
author: Unreal
date: '2021-04-27 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-17/
---

## Collision 살펴보기

임포트했던 IcoSpere를 더블클릭한다.

![](/assets/img/posts/Unreal/Basic-17-1.PNG){:class="img-fluid"}

콜리전을 열어본다.

![](/assets/img/posts/Unreal/Basic-17-2.PNG){:class="img-fluid"}

콜리전은 콜리전 탭에서 추가/제거도 가능하다

![](/assets/img/posts/Unreal/Basic-17-3.PNG){:class="img-fluid"}

---

## 피직스 항목을 Enable시킨다.

![](/assets/img/posts/Unreal/Basic-17-4.PNG){:class="img-fluid"}

---

## 콜리전을 넣고 실행시 물체가 바닥에 떨어진다.

콜리전의 개수에 따라 자연스럽게 떨어진다.

![](/assets/img/posts/Unreal/Basic-17-5.PNG){:class="img-fluid"}

---

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Floater.generated.h"

UCLASS()
class BASIC_API AFloater : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AFloater();

	UPROPERTY(VisibleAnywhere, Category = "ActorMeshComponents")
	UStaticMeshComponent* StaticMesh;

	// Location use by SetActorLocation() when BegniPlay() is called
	UPROPERTY(EditInstanceOnly, BlueprintReadWrite, Category = "FloaterVectors")
	FVector InitialLocation;

	// Location of the Actor when dragged in from the editor
	UPROPERTY(VisibleInstanceOnly, BlueprintReadWrite, Category = "FloaterVectors")
	FVector PlacedLocation;

	UPROPERTY(VisibleDefaultsOnly, BlueprintReadOnly, Category = "FloaterVectors")
	FVector WorldOrigin;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "FloaterVectors")
	FVector InitialDirection;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "FloaterVectors")
	bool bShouldFloat;

	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "Floater Variables")
	bool bInitializerFloaterLocations;

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

};

```

```cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "Floater.h"

// Sets default values
AFloater::AFloater()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	StaticMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("CustomStaticMesh"));

	InitialLocation = FVector(0.0f);
	PlacedLocation = FVector(0.0f);
	WorldOrigin = FVector(0.0f);

	InitialDirection = FVector(0.0f);

	bInitializerFloaterLocations = false;
	bShouldFloat = false;
}

// Called when the game starts or when spawned
void AFloater::BeginPlay()
{
	Super::BeginPlay();
	
	//SetActorLocation(FVector(0.0f, 0.0f, 0.0f));

	PlacedLocation = GetActorLocation();

	if (bInitializerFloaterLocations)
	{
		SetActorLocation(InitialLocation);
	}
}

// Called every frame
void AFloater::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if (bShouldFloat)
	{
		FHitResult HitResult;
		AddActorLocalOffset(InitialDirection, false, &HitResult);
	}
}


```