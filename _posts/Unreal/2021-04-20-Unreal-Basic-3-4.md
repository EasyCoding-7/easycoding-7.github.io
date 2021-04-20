---
layout: post
title:  "(Unreal : Basic) 3-4 : Creating a UObject"
summary: ""
author: Unreal
date: '2021-04-20 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-3-4/
---

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "MyObject.generated.h"

/**
 * 
 */
UCLASS()
class FIRSTPROJECT_API UMyObject : public UObject
{
	GENERATED_BODY()
	
public:
	UMyObject();

private:
	float MyFloat;

	void MyFunction();
};
```

```cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "MyObject.h"

UMyObject::UMyObject()
{
	MyFloat = 0.f;
}

void UMyObject::MyFunction()
{

}
```

## 만약 C++ Object에 BluePrint를 쓰고싶다면??

```cpp
UCLASS(Blueprintable)
class FIRSTPROJECT_API UMyObject : public UObject
{
	GENERATED_BODY()
	
public:
	UMyObject();

    // ...
```

![](/assets/img/posts/Unreal/basic-3-4-1.PNG){:class="img-fluid"}

Blueprint를 사용할수 있음을 확인할 수 있다.<br>
실제로 Blueprint를 사용해보자면

![](/assets/img/posts/Unreal/basic-3-4-2.PNG){:class="img-fluid"}

이제 Blueprint에서 C++ Object의 변수에 접근해보자.

```cpp
UCLASS(Blueprintable)
class FIRSTPROJECT_API UMyObject : public UObject
{
	GENERATED_BODY()
	
public:
	UMyObject();

public:
	UPROPERTY(BlueprintReadWrite)
	float MyFloat;

	UFUNCTION(BlueprintCallable)
	void MyFunction();
};
```

![](/assets/img/posts/Unreal/basic-3-4-3.PNG){:class="img-fluid"}
