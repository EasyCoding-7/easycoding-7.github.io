---
layout: post
title:  "(Unreal : Basic) 3-5 : Using UObject in Blueprints"
summary: ""
author: Unreal
date: '2021-04-20 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-3-5/
---

```cpp
UCLASS(Blueprintable)
class FIRSTPROJECT_API UMyObject : public UObject
{
	GENERATED_BODY()
	
public:
	UMyObject();

public:
	UPROPERTY(BlueprintReadWrite, Category = "MyVariables")
	float MyFloat;

	UFUNCTION(BlueprintCallable, Category = "MyFunctions")
	void MyFunction();
};
```

![](/assets/img/posts/Unreal/basic-3-5-1.PNG){:class="img-fluid"}

카테고리를 지정할 수 있다.

## Text Message 출력시켜보기

```cpp
// 코드 추가
void UMyObject::MyFunction()
{
	UE_LOG(LogTemp, Warning, TEXT("This is our warning text!"));
}
```

![](/assets/img/posts/Unreal/basic-3-5-2.PNG){:class="img-fluid"}

1. 새로운 변수를 추가
2. 변수를 MyObject로 상속
3. BeginPlay시에 MyObject의 함수를 연결

그냥 실행하면 Object를 생성하지 않았기에 실행이 안됨 아래와 같이 실행하자

![](/assets/img/posts/Unreal/basic-3-5-3.PNG){:class="img-fluid"}