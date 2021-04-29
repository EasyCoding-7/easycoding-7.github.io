---
layout: post
title:  "(Unreal : Basic) 31. Pawn Camera Rotation"
summary: ""
author: Unreal
date: '2021-04-28 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-31/
---

## 카메라 Pitch, Yaw 매핑

![](/assets/img/posts/Unreal/Basic-31-1.PNG){:class="img-fluid"}

```cpp
void ACollider::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PlayerInputComponent->BindAxis(TEXT("MoveForward"), this, &ACollider::MoveForward);
	PlayerInputComponent->BindAxis(TEXT("MoveRight"), this, &ACollider::MoveRight);

	PlayerInputComponent->BindAxis(TEXT("CameraPitch"), this, &ACollider::PitchCamera);
	PlayerInputComponent->BindAxis(TEXT("CameraYaw"), this, &ACollider::YawCamera);
}

void ACollider::PitchCamera(float input)
{
	CameraInput.Y = input;
}

void ACollider::YawCamera(float input)
{
	CameraInput.X = input;
}
```

```cpp
void ACollider::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	FRotator NewRotation = GetActorRotation();
	NewRotation.Yaw += CameraInput.X;
	SetActorRotation(NewRotation);

	FRotator NewSpringArmRotation = SpringArm->GetComponentRotation();
	NewSpringArmRotation.Pitch += CameraInput.Y;

	SpringArm->SetWorldRotation(NewSpringArmRotation);
}
```