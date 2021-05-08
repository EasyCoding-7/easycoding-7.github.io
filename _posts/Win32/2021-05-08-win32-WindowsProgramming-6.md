---
layout: post
title:  "(Win32 : WindowsProgramming-6) C++과 MASM"
summary: ""
author: win32
date: '2021-05-08 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-6/
---

## C++의 Name Mangling

```cpp
// main.cpp

int main()
{
    square(3);
    // Error - squre를 못찾음.
}
```

```cpp
// square.h
#pragma once

int square(int a);
```

```cpp
// square.c

int square(int a)
{
    return a * a;
}
```

Why?

일단 비교를 위해 두 가지 방법으로 square.c를 컴파일 해보자.

```
$ cl /Tc squre.c /c /FAs /Faasm1.asm
# /Tc : c 스타일로 컴파일
# /c : 컴파일만 해달라
# /FAs filename : filename으로 어셈블리제작
$ cl /Tp squre.c /c /FAs /Faasm2.asm
# /Tp : c++ 스타일로 컴파일
```

이제 두 개의 어셈블리를 비교해보자면 함수의 이름이 다르다.

```
// C
_squre

// C++
?squre@@YAHH@Z
```

C++은 함수오버로딩(매개변수를 다르게 넣음)으로 함수의 이름이 하나로 통일될 수 없다.<br>
이 부분이 C/C++의 호환성에 문제가 발생, Name Mangling이라 한다.

---

## 그렇다면 위 예제에서 에러가 발생한 이유는?

* square.c를 c문법으로 컴파일 진행
* main.cpp은 c++문법으로 컴파일 진행
* main.cpp에서 square은 `?squre@@YAHH@Z` 이런식
* main기준에선 square를 찾을 수 없다.

---

## 해결법?

```cpp
// c문법이라고 알려준다.

// square.h
#pragma once

extern "C" int square(int a);
```

---

## 또 다른 문제?

main.cpp -> main.c로 변경하면?? 또 square를 찾을수 없다고 나온다.

Why? -> `extern "C"`자체를 C++에서 구현이 안됨

### 해결책?

```cpp
// square.h
#pragma once

#ifdef __cplusplus
extern "C" {
#endif

int square(int a);

#ifdef __cplusplus
}
#endif
```

---

## 그렇다면 어셈블리와 Name Mangling현상은 없을까?

```cpp
#include <stdio.h>

int asm_main();

int main()
{
    int n = asm_main();     // call ?asm_main@...
    // Error 발생
    printf("result : %d\n", n);
}
```

```
.model flat

public _asm_main

.code

_asm_main:
    mov eax, 100
    ret

end
```

## 해결책

1. extern "C" 사용

```cpp
#include <stdio.h>

extern "C" int asm_main();

int main()
{
    int n = asm_main();
    printf("result : %d\n", n);
}
```

2. 어셈블리 함수 이름을 변경

```
.model flat

public ?_asm_main@@YAHXZ

.code

?_asm_main@@YAHXZ:
    mov eax, 100
    ret

end
```