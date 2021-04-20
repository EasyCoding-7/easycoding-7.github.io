---
layout: post
title:  "(Unreal : Basic) 3-2 : Class Creation in Unreal Engine"
summary: ""
author: Unreal
date: '2021-04-20 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-3-2/
---

![](/assets/img/posts/Unreal/basic-3-2-1.PNG){:class="img-fluid"}

![](/assets/img/posts/Unreal/basic-3-2-2.PNG){:class="img-fluid"}

대략 코드는 아래와 같고 ... <br>
중점적으로 살펴봐야 할 부분은

```cpp
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "MainCharacter.generated.h"

UCLASS()
// 1. UCLASS라는 매크로를 통해 클래스를 정의한다
class FIRSTPROJECT_API AMainCharacter : public ACharacter 
// 2. ACharacter의 상속을 받는다
{
	GENERATED_BODY()

public:
	// Sets default values for this character's properties
	AMainCharacter();

protected:
	// Called when the game starts or when spawned
    // 3. begin, tick, setup을 virtual로 만든다
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

};
```

그럼 ACharacter가 뭔지 좀 더 살펴보자면

```cpp
UCLASS(config=Game, BlueprintType, meta=(ShortTooltip="A character is a type of Pawn that includes the ability to walk around."))
// APawn을 상속
class ENGINE_API ACharacter : public APawn
{
	GENERATED_BODY()
```

이런식의 Unreal Engine Hierachy가 있음을 기억하자

