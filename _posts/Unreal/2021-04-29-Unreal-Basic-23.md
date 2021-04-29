---
layout: post
title:  "(Unreal : Basic) 23. The Sine Function"
summary: ""
author: Unreal
date: '2021-04-29 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-23/
---

```cpp
void AFloater::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if (bShouldFloat)
	{
		FVector NewLocation = GetActorLocation();
		NewLocation.Z = NewLocation.Z + FMath::Sin(RunningTime);


		SetActorLocation(NewLocation);
		RunningTime += DeltaTime;
	}
}
```

Floater가 위/아래로 움직인다.

좀 더 테스트하기 쉽게만들기 

```cpp
void AFloater::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if (bShouldFloat)
	{
		FVector NewLocation = GetActorLocation();
		NewLocation.Z = BaseZLocation + Amplitude * FMath::Sin(RunningTime);


		SetActorLocation(NewLocation);
		RunningTime += DeltaTime;
	}
}
```

![](/assets/img/posts/Unreal/Basic-23-1.PNG){:class="img-fluid"}

```cpp
	// Amplitude
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Floater Variables")
	float A{};

	// Period
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Floater Variables")
	float B{};

	// Phase Shift
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Floater Variables")
	float C{};

	// Vertical Shift
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Floater Variables")
	float D{};
```

```cpp
void AFloater::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if (bShouldFloat)
	{
		FVector NewLocation = GetActorLocation();
		NewLocation.Z = BaseZLocation + A * Amplitude * FMath::Sin(B * RunningTime - C) + D;

		SetActorLocation(NewLocation);
		RunningTime += DeltaTime;
	}
}
```

![](/assets/img/posts/Unreal/Basic-23-2.PNG){:class="img-fluid"}