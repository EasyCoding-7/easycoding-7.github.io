---
layout: post
title:  "(Unreal) 두 Location 사이 거리 구하기"
summary: ""
author: Unreal
date: '2021-05-04 0:00:00 +0000'
category: ['Unreal']
#tags: ['Unreal Engine C++ Developer: Learn C++ and Make Video Games']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/GetDistance/
---

```cpp
flaot distance = FVector::Dist(Pawn->GetActorLocation(), GetActorLocation());
```

