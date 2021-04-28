---
layout: post
title:  "(Unreal : Basic) 25. Pawn Class"
summary: ""
author: Unreal
date: '2021-04-27 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-25/
---

## Pawn 클래스 생성

![](/assets/img/posts/Unreal/Basic-25-1.PNG){:class="img-fluid"}

## Pawn Class 기본 코드

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Pawn.h"
#include "Critter.generated.h"

UCLASS()
class BASIC_API ACritter : public APawn
{
	GENERATED_BODY()

public:
	// Sets default values for this pawn's properties
	ACritter();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

};
```

```cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "Critter.h"

// Sets default values
ACritter::ACritter()
{
 	// Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

}

// Called when the game starts or when spawned
void ACritter::BeginPlay()
{
	Super::BeginPlay();
	
}

// Called every frame
void ACritter::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

// Called to bind functionality to input
void ACritter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

}
```

---

## StaticMesh, Camera를 지정해 준다.

```cpp
UCLASS()
class BASIC_API ACritter : public APawn
{
    // ...

    UPROPERTY(EditAnywhere, Category = "Mesh")
	class UStaticMeshComponent* MeshComponent;

	UPROPERTY(EditAnywhere)
	class UCameraComponent* Camera;
};
```

```cpp
#include "Critter.h"
#include <Engine/Classes/Components/StaticMeshComponent.h>
#include <Engine/Classes/Camera/CameraComponent.h>

ACritter::ACritter()
{
 	// ...

	RootComponent = CreateDefaultSubobject<USceneComponent>(TEXT("RootComponent"));

	// Mesh 지정
	MeshComponent = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComponent"));
	MeshComponent->SetupAttachment(GetRootComponent());

	// Camera 지정
	Camera = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
	Camera->SetupAttachment(GetRootComponent());
	Camera->SetRelativeLocation(FVector(-300.0f, 0.f, 300.f));
	Camera->SetRelativeRotation(FRotator(-45.f, 0.f, 0.f));
}
```

---

## Blueprint를 연결한다.

![](/assets/img/posts/Unreal/Basic-25-2.PNG){:class="img-fluid"}

![](/assets/img/posts/Unreal/Basic-25-3.PNG){:class="img-fluid"}

Blueprint 에 StaticMesh를 지정한다.

![](/assets/img/posts/Unreal/Basic-25-4.PNG){:class="img-fluid"}

지정한 카메라의 위치를 확인한다.

![](/assets/img/posts/Unreal/Basic-25-5.PNG){:class="img-fluid"}

---

## GameMode를 Blueprint에 연결한다.

![](/assets/img/posts/Unreal/Basic-25-6.PNG){:class="img-fluid"}

![](/assets/img/posts/Unreal/Basic-25-7.PNG){:class="img-fluid"}

Default Pawn을 Critter_BP로 변경한다.

![](/assets/img/posts/Unreal/Basic-25-8.PNG){:class="img-fluid"}

```cpp
ACritter::ACritter()
{
 	// ...

	AutoPossessPlayer = EAutoReceiveInput::Player0;
}
```

월드 세팅의 게임모드를 CritterGameMode_BP로 변경한다.

![](/assets/img/posts/Unreal/Basic-25-9.PNG){:class="img-fluid"}

여기까지하면 Critter_BP를 기준으로 한 게임을 세팅 끝(이제 시작이다...)