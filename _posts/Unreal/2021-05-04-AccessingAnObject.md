---
layout: post
title:  "(Unreal) Accessing An Object's Name"
summary: ""
author: Unreal
date: '2021-05-04 0:00:00 +0000'
category: ['Unreal']
#tags: ['Unreal Engine C++ Developer: Learn C++ and Make Video Games']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/AccessingAnObject/
---

```cpp
FString ObjectName = GetOwner()->GetName();
UE_LOG(LogTemp, Warning, TEXT("Object name : %s"), *ObjectName);
// Unreal에서 지정한 이름이 나온다.
```
