---
layout: post
title:  "(Unreal : Tutorial) 7. 프로그래밍 퀵스타트"
summary: ""
author: Unreal
date: '2021-04-26 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Tutorial-7/
---

* [참고사이트](https://www.youtube.com/watch?v=rI9auiWitYA&list=PLYQHfkihy4AxmwLN7Tn_958qChILAynw_&index=7)

여기는 쉬워서 새로운 부분만 별도 정리

## Actor 위치 이동시키기

```cpp
void AActor::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);

    FVector NewLocation = GetActorLocation();
    float DeltaHeight = (FMath::Sin(sunningTime + DeltaTime) - FMath::Sin(RunningTime));
    NewLocation.Z += DeltaHeight * 20.0f;
    RunningTime += DeltaTime;
    SetActorLocation(NewLocation);
}
```
