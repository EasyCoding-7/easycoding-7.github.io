---
layout: post
title:  "(Unreal : Tutorial) 9. 변수 UPROPERTY"
summary: ""
author: Unreal
date: '2021-04-27 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Tutorial-9/
---

* [참고사이트](https://www.youtube.com/watch?v=SGrH9vZUWDk&list=PLYQHfkihy4AxmwLN7Tn_958qChILAynw_&index=9)

---

## Unreal에서 사용되는 변수

```cpp
// unreal은 short, int, long 대신 아래를 사용
int8 i8;
int16 i16;
int32 i32;		
int64 i64;

// unsinged 표현은
uint8 ui8;

// 소수표현
float f;
double d;

// 문자열 표현
FString fs; // TEXT 매크로로 문자열을 넣는다

// 논리 변수
bool b1;
```

---

## Example

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Damage")
int32 TotalDamage;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Damage")
float DamageTimeInSeconds;

UPROPERTY(BlueprintReadOnly, VisibleAnywhere, Transient, Category = "Damage")
float DamagePerSecond;

UPROPERTY(EditAnywhere, BlueprintReadWrite)
FString CharacterName;

UPROPERTY(EditAnywhere, BlueprintReadWrite)
bool bAttackable;
```

![](/assets/img/posts/Unreal/tutorial-9-1.PNG){:class="img-fluid"}