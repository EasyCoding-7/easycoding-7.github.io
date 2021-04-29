---
layout: post
title:  "(Unreal : Basic) 19. Sweeping"
summary: ""
author: Unreal
date: '2021-04-29 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-19/
---

실험을 위해 벽을 하나 만들어보자.

![](/assets/img/posts/Unreal/Basic-19-1.PNG){:class="img-fluid"}

이후 Floater하나를 생성 후 X 방향(벽이있는 방향)으로 흐르게 선언한다면

![](/assets/img/posts/Unreal/Basic-19-2.PNG){:class="img-fluid"}

그냥 벽을 통과해버릴 것이다.

![](/assets/img/posts/Unreal/Basic-19-3.PNG){:class="img-fluid"}

여기서 벽의 피직스를 켜줘야 그냥 통과하지 못한다.

---

다른방법으로는 우선 벽의 피직스를 끄고 코드를 아래와 같이 수정해보자.

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

추가적으로 충돌이 발생시 충돌지점을 아래와 같이 출력이 가능하다.
