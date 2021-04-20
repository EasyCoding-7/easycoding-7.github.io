---
layout: post
title:  "(Unreal : Basic) 4-3 : F-Vectors 1"
summary: ""
author: Unreal
date: '2021-04-20 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-4-3/
---

## C++에서 Vector반영하기

```cpp
// Called when the game starts or when spawned
void AFloater::BeginPlay()
{
	Super::BeginPlay();
	
	SetActorLocation(FVector(0.0f, 0.0f, 0.0f));
}
```

![](/assets/img/posts/Unreal/basic-4-3-1.PNG){:class="img-fluid"}