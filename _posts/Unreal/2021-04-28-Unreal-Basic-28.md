---
layout: post
title:  "(Unreal : Basic) 28. Pawn Movement Input-3"
summary: ""
author: Unreal
date: '2021-04-28 0:00:00 +0000'
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