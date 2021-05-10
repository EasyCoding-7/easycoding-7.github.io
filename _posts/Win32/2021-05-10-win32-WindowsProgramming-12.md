---
layout: post
title:  "(Win32 : WindowsProgramming-12) Dynamic Link Library(DLL)"
summary: ""
author: win32
date: '2021-05-10 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-12/
---

## DLL 만들기

```cpp
// mymath.c

int add(int a, int b)
{
    return a + b;
}

int __stdcall sub(int a, int b)
{
    return a - b;
}
```

```
$ cl mymath.c /c
$ link mymath.obj /DLL

# or

$ cl mymath.c /LD
```

---

## Symbol Export

1. 함수 앞에 __declspec(dllexport) 지시어 추가
2. 모듈 정의 파일(.def) 사용(나중에 정리)

```cpp
// mymath.c

__declspec(dllexport) int add(int a, int b)
{
    return a + b;
}

__declspec(dllexport) int __stdcall sub(int a, int b)
{
    return a - b;
}
```

심볼을 export할 시 dll만 만들어지는게 아니라 lib도 같이 만들어지게 된다.<br>
lib는 함수의 기계어 코드를 담거나 혹은 export되는 함수의 링크 정보를 담고 있는다.

---

## 배포

```cpp
// mymath.h
#pragma once

#ifdef __cplusplus
extern "C" {
#endif

    __declspec(dllimport) int add(int a, int b);
    __declspec(dllimport) int sub(int a, int b);

#ifdef __cplusplus
}
#endif
```

---

## DLL 사용하기

```cpp
#include <stdio.h>
#include <mymath.h>

#pragma comment(lib, "mydynamic.lib")

int main()
{
    int ret = add(1, 2)
    printf("result  %d\n", ret);
}
```

---

## CRT 함수와 DLL

```cpp
#include <stdio.h>
#include <Windows.h>

int main()
{
    MeesageBoxA(0, "A", "B", MB_OK);
    // Windows API는 DLL로 제공이 된다.
    // 그럼 dll, lib, h의 추가는 어디서 진행이 될까?

    printf("hello\n");
}
```

* 일단 .h는 `#include <Windows.h>`
* lib는? 프로젝트 속성 -> 링커에 디폴트로 추가 되어있다.

그럼 `printf`는?<br>
이런 C표준 함수르 CRT함수라 한다.<br>
CRT함수는 정적/동적 라이브러리를 모두 제공한다.

* 정적라이브러리(libucrt.lib) : `/MT`
* 정적라이브러리 디버그(libucrtd.lib) : `/MTd`
* 동적라이브러리(ucrtbase.dll) : `/MD`
* 동적라이브러리 디버그(ucrtbased.dll) : `/MDd`

일반적으로 IDE를 쓰면 동적라이브러리 버전을 사용하게 됨.

---

## CRT 라이브러리의 충돌

```cpp
// main.exe

int main()
{
    void* p = Alloc(100)l
    free(p);
    // main.exe에서는 CRT 동적 라이브러리 사용
}
```

```cpp
// Sample.dll

void * Alloc(int size)
{
    return malloc(size);
    // Sample.dll에서는 CRT 정적 라이브러리 사용
}
```

* CRT 라이브러리 버전을 맞추어야 한다
* DLL 내에서 자원 할당 한 경우 DLL 에서 자원 해지하는 것이 좋다.

---

## Explicit Linking(DLL 명시적 연결)

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

#include "mydynamic.h"

#pragma comment(lib, "mydynamic.lib")

int main()
{
    int ret = add(1, 2);
    printf("result : %d\n", ret);
}
```

* 암시적 연결(implicit linking) : 실행파일 실행 시 동시에 DLL에 Load
* 명시적 연결(explicit linking) : DLL이 필요할때 `LoadLibrary` 함수를 사용해서 DLL 로드

```cpp
#include <stdio.h>
#include <Windows.h>
#include <tchar.h>

typedef int (*F)(int, int);

int main()
{
    getchar();
    // 1. DLL 로드
    HMODULE hDll = LoadLibrary(_T("MyDynamic.dll"));
    printf("DLL 주소 : %p\n", hDll);
    // 2. DLL 안에서 함수 찾기
    F f = (F)GetProcAddress(hDll, "add");
    printf("함수 주소 : %p\n", f);

    int ret = f(1, 2);
    printf("result : %d\n", ret);

    // 3. DLL 언로드
    FreeLibrary(hDll);
}
```

```cpp
// 단, 콜링 컨벤션으로 인해서
F f = (F)GetProcAddress(hDll, "sub");       // 못찾음
F f2 = (F)GetProcAddress(hDll, "_sub@8");   // 찾음
printf("함수 주소 : %p\n", f);               // 0
printf("함수 주소 : %p\n", f2);

int ret = f2(1, 2);             // Error - 콜링 컨벤션이 달라 메모리해지 안됨.
printf("result : %d\n", ret);
```

```cpp
typedef int (__stdcall *F2)(int, int);

F2 f2 = (F2)GetProcAddress(hDll, "_sub@8");

int ret = f2(1, 2);     // ok
```