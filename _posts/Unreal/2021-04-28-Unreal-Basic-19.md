---
layout: post
title:  "(Unreal : Basic) 19. Sweeping"
summary: ""
author: Unreal
date: '2021-04-28 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-19/
---

```cpp
void AFloater::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if (bShouldFloat)
	{
		FHitResult HitResult;
		// AddActorLocalOffset(InitialDirection, false, &HitResult);
        AddActorLocalOffset(InitialDirection, true, &HitResult);
        // 다른 object를 통과하지 못한다.

        FVector HitLocation = HitResult.Location;
        UE_LOG(LogTemp, Warning, TEXT("Hit Location: x = %f, y = %f, z = %f"), HitLocation.X, HitLocation.Y, HitLocation.Z);
	}
}
```

