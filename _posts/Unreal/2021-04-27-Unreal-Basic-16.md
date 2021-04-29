---
layout: post
title:  "(Unreal : Basic) 16 : F-Vectors 3"
summary: ""
author: Unreal
date: '2021-04-27 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-16/
---

![](/assets/img/posts/Unreal/basic-4-5-1.PNG){:class="img-fluid"}

`FVector WorldOrigin;`을 추가

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

	bInitializerFloaterLocations = false;
}

// Called when the game starts or when spawned
void AFloater::BeginPlay()
{
	Super::BeginPlay();
	
	SetActorLocation(FVector(0.0f, 0.0f, 0.0f));

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

}
```

![](/assets/img/posts/Unreal/basic-16-2.PNG){:class="img-fluid"}

Blueprint에서 값을 변경할 수 없음을 확인

```cpp
AFloater::AFloater()
{
 	// ...

	InitialDirection = FVector(0.0f);

	bInitializerFloaterLocations = false;
	bShouldFloat = false;
}

// ...

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