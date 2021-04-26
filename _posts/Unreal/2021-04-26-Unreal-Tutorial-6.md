---
layout: post
title:  "(Unreal : Tutorial) 6. 로그 출력"
summary: ""
author: Unreal
date: '2021-04-26 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Tutorial-6/
---

* [참고사이트](https://www.youtube.com/watch?v=rI9auiWitYA&list=PLYQHfkihy4AxmwLN7Tn_958qChILAynw_&index=6)

---

## 로그 출력 화면 보기

창 -> 개발자 툴 -> 출력 로그

![](/assets/img/posts/Unreal/tutorial-6-1.PNG){:class="img-fluid"}

---

## 로그 출력하기

```cpp
void AMyActor::BeginPlay()
{
	Super::BeginPlay();

	UE_LOG(LogTemp, Log, TEXT("Log Message"));	
}
```

![](/assets/img/posts/Unreal/tutorial-6-2.PNG){:class="img-fluid"}

---

## 좀 더 다양한 로그형태 사용

```cpp
void AMyActor::BeginPlay()
{
	Super::BeginPlay();

	UE_LOG(LogTemp, Error, TEXT("Error Message"));	
    UE_LOG(LogTemp, Warning, TEXT("Warning Message"));	
    UE_LOG(LogTemp, Display, TEXT("Display Message"));	
}
```

![](/assets/img/posts/Unreal/tutorial-6-3.PNG){:class="img-fluid"}

---

## 다양한 변수 출력

```cpp
void AMyActor::BeginPlay()
{
	Super::BeginPlay();

    FString CharacterName = TEXT("HiWer");
    UE_LOG(LogTemp, Log, TEXT("Charater Name = %s"), *CharacterName);

    bool isAttackable = ture;
    UE_LOG(LogTemp, Log, TEXT("isAttackable = %s"), isAttackable ? TEXT("true") : TEXT("false"));

    int hp = 100;
    UE_LOG(LogTemp, Log, TEXT("HP = %d"), hp);

    float AttackSpeed = 1.0f
    UE_LOG(LogTemp, Log, TEXT("AttackSpeed = %f"), AttackSpeed);

    FVector CharacterPosition = GetActorLocation();
    UE_LOG(LogTemp, Log, TEXT("CharacterPosition = %s"), CharacterPosition.ToString());
}
```



