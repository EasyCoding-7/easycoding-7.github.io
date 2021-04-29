---
layout: post
title:  "(Unreal : Basic) 39. The Animation Blueprint"
summary: ""
author: Unreal
date: '2021-04-29 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-39/
---

## Blueprint 애니메이션 생성

![](/assets/img/posts/Unreal/Basic-39-1.PNG){:class="img-fluid"}

![](/assets/img/posts/Unreal/Basic-39-2.PNG){:class="img-fluid"}

서있다가 달려가는 애니메이션을 만들어보자.

![](/assets/img/posts/Unreal/Basic-39-3.PNG){:class="img-fluid"}

1. idle을 넣고
2. run을 넣는다.

## 캐릭에 적용해보자.

![](/assets/img/posts/Unreal/Basic-39-4.PNG){:class="img-fluid"}

애니메이션 블루프린트 생성

![](/assets/img/posts/Unreal/Basic-39-5.PNG){:class="img-fluid"}

![](/assets/img/posts/Unreal/Basic-39-6.PNG){:class="img-fluid"}

일단 이렇게 만들기도 가능하다고 알고만 있고 실제는 C++로 제작예정

![](/assets/img/posts/Unreal/Basic-39-7.PNG){:class="img-fluid"}

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "Animation/AnimInstance.h"
#include "MainAnimInstance.generated.h"

/**
 * 
 */
UCLASS()
class BASIC_API UMainAnimInstance : public UAnimInstance
{
	GENERATED_BODY()

public:
	virtual void NativeInitializeAnimation() override;

    UFUNCTION(BlueprintCallable, Category = AnmiationProperties)
	void UpdateAnimationProperties();

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Movement)
	float MovementSpeed;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Movement)
	bool bIsInAir;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Movement)
	class APawn* Pawn;
};

```

```cpp
#include "MainAnimInstance.h"
#include <Engine/Classes/GameFramework/CharacterMovementComponent.h>

void UMainAnimInstance::NativeInitializeAnimation()
{
	if (Pawn == nullptr)
	{
		Pawn = TryGetPawnOwner();
	}


}

void UMainAnimInstance::UpdateAnimationProperties()
{
	if (Pawn == nullptr)
	{
		Pawn = TryGetPawnOwner();
	}

	if (Pawn)
	{
		FVector Speed = Pawn->GetVelocity();
		FVector LateralSpeed = FVector(Speed.X, Speed.Y, 0.f);
		MovementSpeed = LateralSpeed.Size();

		bIsInAir = Pawn->GetMovementComponent()->IsFalling();
	}
}

```

## MainAnimInstance를 Blueprint로

![](/assets/img/posts/Unreal/Basic-39-8.PNG){:class="img-fluid"}

![](/assets/img/posts/Unreal/Basic-39-9.PNG){:class="img-fluid"}