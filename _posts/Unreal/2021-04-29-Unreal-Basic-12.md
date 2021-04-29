---
layout: post
title:  "(Unreal : Basic) 12 : Actors and Actor Components"
summary: ""
author: Unreal
date: '2021-04-29 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-12/
---

여기서 부터 강의 시작이라 생각해도 좋음.<br>
하나하나 다 따라와야함 뒤에서 다 사용됨.

## 새로운 Actor C++ 클래스를 만들어보자.

![](/assets/img/posts/Unreal/basic-4-1-1.PNG){:class="img-fluid"}

![](/assets/img/posts/Unreal/basic-4-1-2.PNG){:class="img-fluid"}

```cpp
// 코드수정해야 빌드됨.
//#include "GamePlayActors/Floater.h"
#include "Floater.h"
```

Floater의 Blueprint 생성

![](/assets/img/posts/Unreal/basic-4-1-3.PNG){:class="img-fluid"}

참고로 AActor의 DefaultSceneRoot(Component)에 다양하게 정의된 Component를 살펴보자

![](/assets/img/posts/Unreal/basic-4-1-4.PNG){:class="img-fluid"}

Cube를 Component로 추가해 보자.

![](/assets/img/posts/Unreal/basic-4-1-5.PNG){:class="img-fluid"}

## StaticMesh를 추가해보자.

```cpp
class FIRSTPROJECT_API AFloater : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AFloater();

	UPROPERTY(VisibleAnywhere, Category = "ActorMeshComponents")
	UStaticMeshComponent* StaticMesh;

    // ...
```

```cpp
AFloater::AFloater()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	StaticMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("CustomStaticMesh"));
}
```

![](/assets/img/posts/Unreal/basic-4-1-6.PNG){:class="img-fluid"}


