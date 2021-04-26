---
layout: post
title:  "(Unreal : Tutorial) 4. 변수, 타이머, 이벤트"
summary: ""
author: Unreal
date: '2021-04-26 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Tutorial-4/
---

* [참고사이트](https://www.youtube.com/watch?v=rI9auiWitYA&list=PLYQHfkihy4AxmwLN7Tn_958qChILAynw_&index=4)

---

```cpp
UCLASS()
class CAMERATUTORIAL_API ACountdown : public AActor
{
	// ...

	int CountdownTime;
	UTextRenderComponent* CountdownText;
	void UpdateTimerDisplay();

    // Timer
	void AdvancedTimer();
	void CountdownHasFinished();
	FTimerHandle CountdownTimerHandle;
};
```

```cpp
ACountdown::ACountdown()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = false;

	CountdownText = CreateDefaultSubobject<UTextRenderComponent>(TEXT("Countdown Text"));
	CountdownText->SetHorizontalAlignment(EHTA_Center);
	CountdownText->SetWorldSize(150.0f);
	RootComponent = CountdownText;

	CountdownTime = 3;
}

void ACountdown::UpdateTimerDisplay()
{
	CountdownText->SetText(FString::FromInt(FMath::Max(CountdownTime, 0)));
}

void ACountdown::AdvancedTimer()
{
	--CountdownTime;
	UpdateTimerDisplay();

	if (CountdownTime < 1)
	{
		GetWorldTimerManager().ClearTimer(CountdownTimerHandle);
		CountdownHasFinished();
	}
}

void ACountdown::CountdownHasFinished()
{
	CountdownText->SetText(TEXT("Go!"));
}

// Called when the game starts or when spawned
void ACountdown::BeginPlay()
{
	Super::BeginPlay();

	UpdateTimerDisplay();
	GetWorldTimerManager().SetTimer(CountdownTimerHandle, this, &ACountdown::AdvancedTimer, 1.0f, true);
}

// ...
```

---

## 변수 Unreal에서도 컨트롤 가능하게 만들기

```cpp
UCLASS()
class CAMERATUTORIAL_API ACountdown : public AActor
{
	// ...

    // 여기 주석을 적으면 언리얼에서도 이 주석이 보임.
    UPROPERTY(EditAnywhere)
	int CountdownTime;
	
    // ...
```

---

## 블루 프린트에서 함수가 보이게 변경

```cpp
UCLASS()
class CAMERATUTORIAL_API ACountdown : public AActor
{
	// ...

    UFUNCTION(BlueprintNativeEvent)
	void CountdownHasFinished();
    virtual void CountdownHasFished_Implementation();

	// ...
```

```cpp
void ACountdown::CountdownHasFished_Implementation()
{
	CountdownText->SetText(TEXT("Go!"));
}
```

---

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include <Engine/Classes/Components/StaticMeshComponent.h>
#include "Door.generated.h"

UCLASS()
class TUTORIALPROJECT_API ADoor : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	ADoor();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	UPROPERTY(EditAnywhere)
	UStaticMeshComponent* DoorMesh;

	UPROPERTY(EditAnywhere)
	float CloseTime;

	float DoorDeltaTime;

	bool bOpen;

	FTimerHandle DorTimerHandle;

	FRotator OriginRotation;

	void Open();
	void Close();
};
```

```cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "Door.h"

// Sets default values
ADoor::ADoor()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;


	DoorMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Door Mesh"));
	RootComponent = DoorMesh;

	CloseTime = 3.0f;
}

// Called when the game starts or when spawned
void ADoor::BeginPlay()
{
	Super::BeginPlay();
	
	OriginRotation = GetActorRotation();
	GetWorldTimerManager().SetTimer(DorTimerHandle, this, &ADoor::Open, 0.03f, true);
}

// Called every frame
void ADoor::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	DoorDeltaTime += DeltaTime;
}

void ADoor::Open()
{
	if (!bOpen)
	{
		bOpen = true;
		DoorDeltaTime = 0.0f;
	}

	FRotator rotation = GetActorRotation();
	rotation = OriginRotation + FRotator(0.0f, FMath::Lerp(0.0f, 90.0f, DoorDeltaTime), 0.0f);
	SetActorRotation(rotation);
	if (DoorDeltaTime > 1.0f)
	{
		GetWorldTimerManager().ClearTimer(DorTimerHandle);
		GetWorldTimerManager().SetTimer(DorTimerHandle, this, &ADoor::Close, 0.03f, true, CloseTime);
	}
}

void ADoor::Close()
{
	if (bOpen)
	{
		bOpen = false;
		DoorDeltaTime = 0.0f;
	}

	FRotator rotation = GetActorRotation();
	rotation = OriginRotation + FRotator(0.0f, FMath::Lerp(90.0f, 00.0f, DoorDeltaTime), 0.0f);
	SetActorRotation(rotation);
	if (DoorDeltaTime > 1.0f)
	{
		GetWorldTimerManager().ClearTimer(DorTimerHandle);
	}
}
```