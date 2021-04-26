---
layout: post
title:  "(Unreal : Tutorial) 5. 헤더 인클루드 시 주의할 점."
summary: ""
author: Unreal
date: '2021-04-26 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Tutorial-5/
---

* [참고사이트](https://www.youtube.com/watch?v=rI9auiWitYA&list=PLYQHfkihy4AxmwLN7Tn_958qChILAynw_&index=5)

---

```cpp
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include <Engine/Classes/Components/StaticMeshComponent.h>
#include "Door.generated.h"

// 항상 .generated.h 위에 선언해야함.
// 아래 선언시 에러발생

UCLASS()
class TUTORIALPROJECT_API ADoor : public AActor
{
	GENERATED_BODY()
```
