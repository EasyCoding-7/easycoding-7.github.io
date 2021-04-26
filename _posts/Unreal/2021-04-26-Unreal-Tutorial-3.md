---
layout: post
title:  "(Unreal : Tutorial) 3. 구조체 만들기"
summary: ""
author: Unreal
date: '2021-04-26 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Tutorial-3/
---

* [참고사이트](https://www.youtube.com/watch?v=rI9auiWitYA&list=PLYQHfkihy4AxmwLN7Tn_958qChILAynw_&index=3)

---

```cpp
USTRUCT(Atomic, BlueprintType)
struct Potion
{
    GENERATED_USTRUCT_BODY()
public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int recoveryHp;
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float cooltime;
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    string potionName;
};
```