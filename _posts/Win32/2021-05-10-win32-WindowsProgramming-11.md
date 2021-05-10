---
layout: post
title:  "(Win32 : WindowsProgramming-11) Static Library"
summary: ""
author: win32
date: '2021-05-10 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-11/
---

## Static Library 만들기

```cpp
// add.c
int add(int a, int b)
{
    return a + b;
}
```

```
# 빌드
$ cl  add.c /c
# 컴파일만 하기에 object(.obj)파일만 생성됨
```

```cpp
// sub.c
int sub(int a, int b)
{
    return a - b;
}
```

```
# 빌드
$ cl sub.c /c
```

add.obj, sub.obj 파일을 하나로 묶어서 배포하고 싶다 -> 정적라이브러리(.lib)

```
# 라이브러리 만들기
$ lib add.obj sub.obj /out:mystaticmanual.lib
```

---

## IDE에서 Static Library 만들기

**프로젝트 속성 -> 일반 -> 구성 형식 -> 정적라이브러리** 로 설정

---

## Static Library 배포

* 헤더파일을 포함해서 배포한다.

```cpp
// mymath.h
#pragma once

#ifdef __cplusplus
extern "C" {
#endif

    int add(int a, int b);
    int sub(int a, int b);

#ifdef __cplusplus
}
#endif
```

---

## 정적 라이브러리 사용하기

```cpp
// main.cpp
#include <stdio.h>
#include "mymath.h"

int main()
{
    int ret = add(1, 2);
    printf("result : %d\n", ret);
}
```

```
$ cl main.cpp /c
$ link main.obj mystaticmanual.lib
```

```
# 더 간단하게는 
$ cl main.c /link mystaticmanual.lib
# or
$ cl main.c mystaticmanual.lib
```

코드에 라이브러리 추가는

```cpp
// main.cpp
#include <stdio.h>
#include "mymath.h"

#pragma comment(lib, "mystaticmanual.lib")

int main()
{
    int ret = add(1, 2);
    printf("result : %d\n", ret);
}
```
