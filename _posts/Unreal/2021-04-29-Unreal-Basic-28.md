---
layout: post
title:  "(Unreal : Basic) 28. Pawn Movement Input-3"
summary: ""
author: Unreal
date: '2021-04-29 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-28/
---

## Collider클래스에 Camera 및 SpringArm 생성

```cpp
UCLASS()
class BASIC_API ACollider : public APawn
{
	// ...

	UPROPERTY(VisibleAnywhere, Category = "Mesh")
	class UCameraComponent* Camera;

	UPROPERTY(VisibleAnywhere, Category = "Mesh")
	class USpringArmComponent* SpringArm;

	FORCEINLINE UStaticMeshComponent* GetMeshComponent() { return MeshComponent; };
	FORCEINLINE void SetMeshComponent(UStaticMeshComponent* Mesh) { MeshComponent = Mesh; }
	FORCEINLINE USphereComponent* GetSphereComponent() { return SphereComponent; };
	FORCEINLINE void SetSphereComponent(USphereComponent* Sphere) { SphereComponent = Sphere; }
	FORCEINLINE UCameraComponent* GetCameraComponent() { return Camera; };
	FORCEINLINE void SetCameraComponent(UCameraComponent* cam) { Camera = cam; }
	FORCEINLINE USpringArmComponent* GetSpringArmComponent() { return SpringArm; };
	FORCEINLINE void SetSpringArmComponent(USpringArmComponent* spring) { SpringArm = spring; }

    // ...

```

```cpp
ACollider::ACollider()
{
 	// ...

	AutoPossessPlayer = EAutoReceiveInput::Player0;

	Camera = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
	Camera->SetupAttachment(GetRootComponent());

	SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
	SpringArm->SetupAttachment(GetRootComponent());
	SpringArm->SetRelativeRotation(FRotator(-45.f, 0.f, 0.f));
	SpringArm->TargetArmLength = 400.f;
	SpringArm->bEnableCameraLag = true;
	SpringArm->CameraLagSpeed = 3.0f;
}
```

![](/assets/img/posts/Unreal/Basic-28-1.PNG){:class="img-fluid"}

조금 조정이 필요하겠군 ...

```cpp
AutoPossessPlayer = EAutoReceiveInput::Player0;

SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
SpringArm->SetupAttachment(GetRootComponent());
SpringArm->SetRelativeRotation(FRotator(-45.f, 0.f, 0.f));
SpringArm->TargetArmLength = 400.f;
SpringArm->bEnableCameraLag = true;
SpringArm->CameraLagSpeed = 3.0f;

Camera = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
Camera->SetupAttachment(SpringArm, USpringArmComponent::SocketName);
```

![](/assets/img/posts/Unreal/Basic-28-2.PNG){:class="img-fluid"}

카메라와 구체와의 거리는 SpringArm의 Length로 조절

![](/assets/img/posts/Unreal/Basic-28-3.PNG){:class="img-fluid"}

---

## 전체코드

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Pawn.h"
#include "Collider.generated.h"

UCLASS()
class BASIC_API ACollider : public APawn
{
	GENERATED_BODY()

public:
	// Sets default values for this pawn's properties
	ACollider();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

	UPROPERTY(VisibleAnywhere, Category = "Mesh")
	class UStaticMeshComponent* MeshComponent;

	UPROPERTY(VisibleAnywhere, Category = "Mesh")
	class USphereComponent* SphereComponent;

	UPROPERTY(VisibleAnywhere, Category = "Mesh")
	class UCameraComponent* Camera;

	UPROPERTY(VisibleAnywhere, Category = "Mesh")
	class USpringArmComponent* SpringArm;

	FORCEINLINE UStaticMeshComponent* GetMeshComponent() { return MeshComponent; };
	FORCEINLINE void SetMeshComponent(UStaticMeshComponent* Mesh) { MeshComponent = Mesh; }
	FORCEINLINE USphereComponent* GetSphereComponent() { return SphereComponent; };
	FORCEINLINE void SetSphereComponent(USphereComponent* Sphere) { SphereComponent = Sphere; }
	FORCEINLINE UCameraComponent* GetCameraComponent() { return Camera; };
	FORCEINLINE void SetCameraComponent(UCameraComponent* cam) { Camera = cam; }
	FORCEINLINE USpringArmComponent* GetSpringArmComponent() { return SpringArm; };
	FORCEINLINE void SetSpringArmComponent(USpringArmComponent* spring) { SpringArm = spring; }

private:
	void MoveForward(float Value);
	void MoveRight(float Value);
};

```

```cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "Collider.h"
#include <Engine/Classes/Components/SphereComponent.h>
#include <Engine/Classes/Camera/Cameracomponent.h>
#include <Engine/Classes/GameFramework/SpringArmComponent.h>

// Sets default values
ACollider::ACollider()
{
 	// Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	RootComponent = CreateDefaultSubobject<USceneComponent>(TEXT("RootComponent"));

	SphereComponent = CreateDefaultSubobject<USphereComponent>(TEXT("SphereComponent"));
	SphereComponent->SetupAttachment(GetRootComponent());
	SphereComponent->InitSphereRadius(40.0f);
	SphereComponent->SetCollisionProfileName(TEXT("Pawn"));

	MeshComponent = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComponent"));
	MeshComponent->SetupAttachment(GetRootComponent());
	static ConstructorHelpers::FObjectFinder<UStaticMesh> MeshComponentAsset(TEXT("StaticMesh'/Game/StarterContent/Shapes/Shape_Cone.Shape_Cone'"));

	if (MeshComponentAsset.Succeeded())
	{
		MeshComponent->SetStaticMesh(MeshComponentAsset.Object);
		MeshComponent->SetRelativeLocation(FVector(0.f, 0.f, -40.f));
		MeshComponent->SetWorldScale3D(FVector(0.8f, 0.8f, 0.8f));
	}

	AutoPossessPlayer = EAutoReceiveInput::Player0;

	SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
	SpringArm->SetupAttachment(GetRootComponent());
	SpringArm->SetRelativeRotation(FRotator(-45.f, 0.f, 0.f));
	SpringArm->TargetArmLength = 400.f;
	SpringArm->bEnableCameraLag = true;
	SpringArm->CameraLagSpeed = 3.0f;

	Camera = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
	Camera->SetupAttachment(SpringArm, USpringArmComponent::SocketName);

	AutoPossessPlayer = EAutoReceiveInput::Player0;
}

// Called when the game starts or when spawned
void ACollider::BeginPlay()
{
	Super::BeginPlay();
	
}

// Called every frame
void ACollider::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

// Called to bind functionality to input
void ACollider::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PlayerInputComponent->BindAxis(TEXT("MoveForward"), this, &ACollider::MoveForward);
	PlayerInputComponent->BindAxis(TEXT("MoveRight"), this, &ACollider::MoveRight);
}

void ACollider::MoveForward(float Value)
{
	FVector Forward = GetActorForwardVector();
	AddMovementInput(Value * Forward);
}

void ACollider::MoveRight(float Value)
{
	FVector Right = GetActorForwardVector();
	AddMovementInput(Value * Right);
}


```