---
layout: post
title:  "(Win32 : WindowsProgramming-4) Calling Convention"
summary: ""
author: win32
date: '2021-05-07 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-4/
---

## 함수 인자 전달방식

```
.model flat
public _asm_main
.code

_asm_main:
    call _add
    ret

_add:   ; _add 호출 시 인자를 넣어보자.
    mov eax, 100
    ret

end
```

* 레지스터에 인자 넣기
* 스택에 인자 넣기

### 레지스터에 인자 넣기

관례적으로 ecx edx, esi, edi 를 인자 레지스터로 많이 쓴다.

```
.model flat
public _asm_main
.code

_asm_main:
    mov edx, 2
    mov ecx, 2
    call _add
    ret

_add:
    mov eax, edx ; eax = edx
    add eax, ecx ; eax += ecx
    ret

end
```

* 장점 : 빠르다
* 단점 : 인자의 개수에 제한이 있다

### 스택에 인자 넣기

```
.model flat
public _asm_main
.code

_asm_main:
    push 2  ; 2번째 인자
    push 1  ; 1번째 인자
    call _add
    ret

_add:
    ; 단순히 pop을 쓸 수 없는게 call은 스택에 돌아올 메모리를 담는다.
    ; 돌아올 주소를 pop해버릴경우 돌아갈 주소를 알 수 없게 된다.
    ret

end
```

* ESP(Extended Stack Pointer) : 가장 최근에 사용한 스택을 가리키는 레지스터
* ESP 기준 4바이트 떨어진 주소값으로 처리해야한다.

```
.model flat
public _asm_main
.code

_asm_main:
    push 2
    push 1
    call _add
    ret

_add:
    mov eax, dword ptr[esp+4]
    add eax, dword ptr[esp+8]
    ; 이렇게 구현하면 에러발생
    ; why? 
    ; esp를 사용하면 esp가 가리키는 주소지 자체가 변경된다.
    ret

end
```

```
2
1
_asm_main 으로 돌아갈 주소 ( <- 현재 esp )
```

```
_add:
    mov eax, dword ptr[esp+4]
    add eax, dword ptr[esp+8]
    ret
    ; ret시에 pop 돌아갈 주소
    ; jmp 꺼낸 주소를 동작하기에
```

사용후

```
2
1 ( <- 현재 esp )
_asm_main 으로 돌아갈 주소
```

esp가 가리키는 주소자체가 변경되게 된다.

* 함수 호출이 종료되면 인자 전달에 사용된 스택은 반드시 파괴되어야 한다.

```
.model flat
public _asm_main
.code

_asm_main:
    push 2
    push 1
    call _add
    add esp, 8 ; esp = esp + 8
    ; 이렇게만 해줘도 스택이 파괴된다.
    ret

_add:
    mov eax, dword ptr[esp+4]
    add eax, dword ptr[esp+8]
    ret

end
```

```
_main으로 돌아갈 주소 ( <- 현재 esp )
2
1
_asm_main 으로 돌아갈 주소
```

---

## 인자 전달과 스택 파괴

파괴하는 방식이 두 가지다.

```
.model flat
public _asm_main
.code

_asm_main:
    push 2
    push 1
    call _add
    add esp, 8  ; (호출자 파괴) 다른 방법으로 파괴할 수 있을까?
    ret

_add:
    mov eax, dword ptr[esp+4]
    add eax, dword ptr[esp+8]
    ret

end
```

```
_add:
    mov eax, dword ptr[esp+4]
    add eax, dword ptr[esp+8]
    ret 8   ; 이렇게 파괴가능 (호출 당한 파괴)

end
```

---

## Calling Convention : C언어는 인자를 어떻게 보낼까?

```cpp
#include <stdio.h>

int add(int a, int b)
{
    int c = a + b;
    return c;
}

int main()
{
    int ret = 0;
    ret = add(1, 2);    // 1, 2는 레지스터로 갈까 스택으로 갈까?

    printf("result : %d\n", ret);
}
```

컴파일러 마다 다르겠지만

```
push 2
push 1      ; 스택에 넣고
call _add   ; 함수호출
add esp, 8  ; 호출한 곳에서 파괴
```

인라인 어셈블로 제작해보자.

```cpp
int main()
{
    int ret = 0;
    
    __asm
    {
        push 2
        push 1
        call add
        add esp, 8
        mov ret, eax
    }

    printf("result : %d\n", ret);
}
```

---

## 정리

```cpp
/*
함수 호출 규약(calling convention)
* 호출 규약에 따라 달라지는 부분은 함수 이름 규칙, 인자 전달 방법, 스택 파괴 위치 이다.
__cdecl : 함수 이름 (_f) / 인자 전달 (스택) / 파괴 위치 (호출자)
__stdcall : 함수 이름 (_f@인자크기) / 인자 전달 (스택) / 파괴 위치 (피호출자)
__fastcall : 함수 이름 (@f@인자크기) / 인자 전달 (2까지 레지스터, 3부터 스택) / 파괴 위치 (3개 이상 피호출자)
*/
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
    * __cdecl : 코드의 길이가 짧아짐
    * __stdcall : 코드의 길이가 길어짐(실행파일의 용량증대, 속도 감소)

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

---

## 어셈블리 언어에서 C함수 호출

```cpp
#include <stdio.h>

int asm_main();

int main()
{
    asm_main();
}

void __cdecl    f1(int a, int b) { printf("f1 : %d, %d\n", a, b); }
void __stdcall  f2(int a, int b) { printf("f2 : %d, %d\n", a, b); }
void __fastcall f3(int a, int b) { printf("f3 : %d, %d\n", a, b); }
void __fastcall f4(int a, int b, int c, int d)
{ 
    printf("f4 : %d, %d, %d, %d\n", a, b, c, d);
}
```

f1, f2, f3, f4를 어셈블리에서 호출해보자.

```
.model flat
public _asm_main

extern _f1: proc
extern _f2@8: proc
extern @f3@8: proc
extern @f4@16: proc

extern _printf: proc
extern _MessageBoxA@16: proc

.data
S1 DB "hello", 10, 0 ; '\n'

.code
_asm_main:
    ; f1(1, 2) 호출
    push 2
    push 1
    call _f1
    add esp, 8

    ; f2(1,2)
    push 2
    push 1
    call _f2@8

    ; f3(1,2)
    mov edx, 2
    mov ecx, 1
    call @f3@8

    ; f4(1,2,3,4)
    push 4
    push 3
    mov edx, 2
    mov ecx, 1
    call @f4@16

    ; 추가) printf 출력
    push offset S1
    call _printf
    add esp, 4

    ; MessageBoxA(0, "Hello", "Hello", MB_OK) 호출
    push 0
    push offset S1
    push offset S1
    call _MessageBoxA@16

    
    ret

end
```