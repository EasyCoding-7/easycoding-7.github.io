---
layout: post
title:  "(C++) 헷깔헷깔시리즈 : 함수 호출 규약(calling convention)"
summary: ""
author: C++
date: '2021-06-11 0:00:00 +0000'
category: ['Cpp', '헷깔헷깔시리즈']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/confusing/calling-convention/
---

## 함수 호출 규약(calling convention)

호출 규약에 따라 달라지는 부분은 함수 이름 규칙, 인자 전달 방법, 스택 파괴 위치 이다.

* `__cdecl` : 함수 이름 (_f) / 인자 전달 (스택) / 파괴 위치 (호출자)
* `__stdcall` : 함수 이름 (_f@인자크기) / 인자 전달 (스택) / 파괴 위치 (피호출자)
* `__fastcall` : 함수 이름 (@f@인자크기) / 인자 전달 (2까지 레지스터, 3부터 스택) / 파괴 위치 (3개 이상 피호출자)

```cpp
void __cdecl f1(int a, int b) {}
void __stdcall f2(int a, int b) {}
void __fastcall f3(int a, int b) {}
void __fastcall f4(int a, int b, int c, int d) {}

int main()  // __cdecl : 디폴트
{
    f1(1, 2);
    f2(1, 2);
    f3(1, 2);
    f4(1, 2, 3, 4);
}
```

```
; __cdecl

	push 2
    push 1
    call _f1
    add esp, 8

; __stdcall

	push 2
    push 1
    call _f2@8

; __fastcall

	mov edx, 2
    mov ecx, 1
    call @f@8

; __fastcall(인자 3개 이상)

	push 4
    push 3
    mov edx, 2
    mov ecx, 1
    call @f4@16
```

* 차이점?
    * 결국 차이점은 스택의 파괴 위치이다.
    * __cdecl : 코드의 길이가 길어짐(실행파일의 용량증대, 속도 감소)
    * __stdcall : 코드의 길이가 짧아짐

* DLL이나 OS에서는 __stdcall을 많이 사용하는데 프로그래밍 언어 사이의 표준이기에 그렇다.

```cpp
void f1(int n, ...) {} // __cdecl

void __stdcall f2(int n, ...) {}

void __fastcall f3(int n, ...) {}

int main()
{
    f1(1, 2);           // add esp, 8
    f1(1, 2, 3, 4);     // add esp, 16

    f2(1, 2);   // 함수내에서 ret8을 해줘야 하는데 가변 매개변수는 ret 몇을 해줘야하는지 알 수 없음
    // 컴파일러가 자동으로 __cdecl로 변경한다.

    f3(1, 2);   // 상동
}
```
