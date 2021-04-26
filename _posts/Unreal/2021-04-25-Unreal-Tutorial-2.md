---
layout: post
title:  "(Unreal : Tutorial) 2. 카메라 조종"
summary: ""
author: Unreal
date: '2021-04-25 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Tutorial-2/
---

* [참고사이트](https://www.youtube.com/playlist?list=PLYQHfkihy4AxmwLN7Tn_958qChILAynw_)

## Project Setting

새 프로젝트 -> 게임

![](/assets/img/posts/Unreal/tutorial-1-1.PNG){:class="img-fluid"}

![](/assets/img/posts/Unreal/tutorial-1-2.PNG){:class="img-fluid"}

---

## 카메라 배치하기

![](/assets/img/posts/Unreal/tutorial-1-3.PNG){:class="img-fluid"}

카메라를 좀 더 쉽게 컨트롤 하는 방법<br>
우클릭 후 파일럿 선택<Br>
카메라가 볼 장면을 지정

![](/assets/img/posts/Unreal/tutorial-1-4.PNG){:class="img-fluid"}

![](/assets/img/posts/Unreal/tutorial-1-5.PNG){:class="img-fluid"}

카메라의 위치가 지정되었음을 확인할 수 있다.

---

## Object가 바라보는 카메라 생성

일단 하나의 OBject를 아무거나 생성

![](/assets/img/posts/Unreal/tutorial-1-6.PNG){:class="img-fluid"}

컴포넌트에 카메라를 추가

![](/assets/img/posts/Unreal/tutorial-1-7.PNG){:class="img-fluid"}

---

## 카메라를 제어할 클래스를 생성해보자.

새 클래스 -> C++ 생성

![](/assets/img/posts/Unreal/tutorial-1-8.PNG){:class="img-fluid"}

![](/assets/img/posts/Unreal/tutorial-1-9.PNG){:class="img-fluid"}

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "CameraControl.generated.h"

UCLASS()
class CAMERATUTORIAL_API ACameraControl : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	ACameraControl();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	UPROPERTY(EditAnywhere)
		AActor* CameraOne;
	UPROPERTY(EditAnywhere)
		AActor* CameraTwo;
	UPROPERTY(EditAnywhere)
		float TimeToNextCameraChange;
};
```

```cpp
#include "CameraControl.h"
#include <Kismet/GameplayStatics.h>

// ...

// Called every frame
void ACameraControl::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	const float TimeBetweenCameraChanges = 2.0f;
	const float SmoothBlendTime = 0.75f;

	TimeToNextCameraChange -= DeltaTime;
	if (TimeToNextCameraChange <= 0.0f)
	{
		TimeToNextCameraChange += TimeBetweenCameraChanges;

		APlayerController* constroller = UGameplayStatics::GetPlayerController(this, 0);
		if (constroller)
		{
			if ((constroller->GetViewTarget() != CameraOne) && (CameraOne != nullptr))
			{
				constroller->SetViewTarget(CameraOne);
			}
			else if ((constroller->GetViewTarget() != CameraTwo) && (CameraTwo != nullptr))
			{
				constroller->SetViewTargetWithBlend(CameraTwo, SmoothBlendTime);
			}
		}
	}
}
```

카메라 컨트롤 클래스를 언리얼에 반영

![](/assets/img/posts/Unreal/tutorial-1-10.PNG){:class="img-fluid"}

카메라1, 카메라2를 지정한다.

![](/assets/img/posts/Unreal/tutorial-1-11.PNG){:class="img-fluid"}

이제 플레이해보면 카메라가 자연스럽게 이동된다.

---

## 움직이는 Object(Actor)에 카메라를 붙여보기

우선 움직이는 Actor를 만들어보자

movingActor 클래스생성

![](/assets/img/posts/Unreal/tutorial-1-12.PNG){:class="img-fluid"}

![](/assets/img/posts/Unreal/tutorial-1-13.PNG){:class="img-fluid"}

```cpp
UCLASS()
class CAMERATUTORIAL_API AMovingActor : public AActor
{
	// ...

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	float Timer;
};
```

```cpp
// Called every frame
void AMovingActor::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	Timer += DeltaTime;

	FVector NewLocation = GetActorLocation();
	NewLocation.Y += FMath::Cos(Timer);
	SetActorLocation(NewLocation);
}
```

MovingActor 클래스를 언리얼에 반영 후 카메라 컴포넌트를 추가

카메라 컨트롤에 MovingActor를 넣어준다.

![](/assets/img/posts/Unreal/tutorial-1-14.PNG){:class="img-fluid"}

---

## 배열로 카메라 받기

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "CameraControl.generated.h"

UCLASS()
class CAMERATUTORIAL_API ACameraControl : public AActor
{
    // ...

    // Unreal에서 배열은 TArray로 받아야 한다.
	UPROPERTY(EditAnywhere)
		TArray<AActor*> Cameras;

	int32 NowCameraIndex;

	//UPROPERTY(EditAnywhere)
	//	AActor* CameraOne;
	//UPROPERTY(EditAnywhere)
	//	AActor* CameraTwo;
	UPROPERTY(EditAnywhere)
		float TimeToNextCameraChange;
};
```

```cpp
void ACameraControl::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	const float TimeBetweenCameraChanges = 2.0f;
	const float SmoothBlendTime = 0.75f;

	TimeToNextCameraChange -= DeltaTime;
	if (TimeToNextCameraChange <= 0.0f)
	{
		AActor* NowGamera = Cameras[NowCameraIndex];

		TimeToNextCameraChange += TimeBetweenCameraChanges;

		APlayerController* constroller = UGameplayStatics::GetPlayerController(this, 0);
		if (constroller)
		{
			//if ((constroller->GetViewTarget() != CameraOne) && (CameraOne != nullptr))
			//{
			//	constroller->SetViewTarget(CameraOne);
			//}
			//else if ((constroller->GetViewTarget() != CameraTwo) && (CameraTwo != nullptr))
			//{
			//	constroller->SetViewTargetWithBlend(CameraTwo, SmoothBlendTime);
			//}

			if ((constroller->GetViewTarget() != NowGamera) && (NowGamera != nullptr))
			{
				constroller->SetViewTarget(NowGamera);
			}
		}

		NowCameraIndex++;
		if (NowCameraIndex >= Cameras.Num())
		{
			NowCameraIndex = 0;
		}
	}
}
```

배열로서 잘 반영됐는지 확인

![](/assets/img/posts/Unreal/tutorial-1-15.PNG){:class="img-fluid"}

---

## 배열을 구조체로 선언

```cpp
//...

USTRUCT(Atomic, BlueprintType)
struct FChangeCameraData
{
	GENERATED_USTRUCT_BODY();
public:
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
		AActor* Camera;
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
		flaot TimeBetweenCameraChanges;
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
		float SmoothBlendTime;
};


UCLASS()
class CAMERATUTORIAL_API ACameraControl : public AActor
{
    // ..

	UPROPERTY(EditAnywhere)
		TArray<FChangeCameraData> Cameras;

	int32 NowCameraIndex;

	UPROPERTY(EditAnywhere)
		float TimeToNextCameraChange;
};
```