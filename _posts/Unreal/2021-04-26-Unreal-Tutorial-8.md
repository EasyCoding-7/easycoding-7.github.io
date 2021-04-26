---
layout: post
title:  "(Unreal : Tutorial) 8. 함수 UFUNCTION"
summary: ""
author: Unreal
date: '2021-04-26 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Tutorial-8/
---

* [참고사이트](https://www.youtube.com/watch?v=rI9auiWitYA&list=PLYQHfkihy4AxmwLN7Tn_958qChILAynw_&index=8)

---

## C++에서 구현한 기능을 Blueprint에서 사용하게 만들기

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyActor.generated.h"

UCLASS()
class TUTORIAL_API AMyActor : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AMyActor();

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Damage")
	int32 TotalDamage;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Damage")
	float DamageTimeInSeconds;

	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "Damage")
	float DamagePerSecond;

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

    // Blueprint에서 사용이 가능하게 선언
	UFUNCTION(BlueprintCallable, Category = "Damage")
	void CalculateDPS();

	virtual void PostInitProperties() override;
	virtual void PostEditChangeProperty(FPropertyChangedEvent& PropertyChangedEvent) override;

};
```

![](/assets/img/posts/Unreal/tutorial-8-1.PNG){:class="img-fluid"}

---

## Blueprint에서 구현한 기능을 C++에서 사용하게 만들기

### 방법1) BlueprintImplementableEvent

```cpp
UFUNCTION(BlueprintImplementableEvent, Category = "Damage")
void CallFromCpp(); // 내부는 Blueprint에서 만들기에 내부는 안만들어도 됨.
```

![](/assets/img/posts/Unreal/tutorial-8-2.PNG){:class="img-fluid"}

### 방법2) BlueprintNativeEvent

```cpp
UFUNCTION(BlueprintNativeEvent, Category = "Damage")
void CallFromCpp();
virtual void CallFromCpp_Implementation();
```

```cpp
void AMyActor::CallFromCpp_Implementation()
{
	UE_LOG(LogTemp, Warning, TEXT("CallFromCpp_Implementation"));
}
```