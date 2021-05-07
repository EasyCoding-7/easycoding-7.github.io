---
layout: post
title:  "(Win32 : WindowsProgramming-5) Stack Frame"
summary: ""
author: win32
date: '2021-05-07 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-5/
---

## esp 레지스터 사용의 위험

```
_add:
    mov eax, dword ptr[esp+4]
    add eax, dword ptr[esp+8]
    ret
```

만약 _add: 함수에 스텍을 사용하는 부분이 있다면??

```
_add:
    push 100
    mov eax, dword ptr[esp+4]
    add eax, dword ptr[esp+8]
    ret
```

esp가 가리키는 주소 값이 바뀐다.

원래의 예상은 여긴데

```
2
1 ( <- esp )
_asm_main으로 돌아갈 주소
```

이렇게 됨.

```
2
1
_asm_main으로 돌아갈 주소
100 ( <- esp )
```

이런 위험을 방지하기 위해서 _asm_main으로 돌아갈 주소값을 별도로 들고있자.

```
_add:
    mov ebp, esp    ; ebp라는 레지스터에 esp를 넣어두자.
    push 100
    mov eax, dword ptr[esp+4]
    add eax, dword ptr[esp+8]
    ret
```

단, 기존에 ebp에 어떤값이 들어가 있을 수 있으니<br>
함수의 호출이 끝날 때, ebp레지스터가 원래 가지고 있는 값을 복구해 주자.

```
_add:
    push ebp        ; ebp의 예전값을 보관
    mov ebp, esp    ; ebp에 esp값을 담고
    push 100
    mov eax, dword ptr[esp+8]
    add eax, dword ptr[esp+12]
    pop ebp         ; ebp값을 복구
    ret
```

```
_main으로 돌아갈 주소
2
1
_asm_main으로 돌아갈 주소
ebp의 예전값 ( <- ebp, esp )
```

결론적으로 추가된 부분이 두 군데 인데

1. ebp를 보관
2. ebp를 복구

이 부분을 Stack Frame이라 한다.

```
_function:
    push ebp        ; ebp의 예전값을 보관
    mov ebp, esp    ; ebp에 esp값을 담고
    ; ...
    pop ebp         ; ebp값을 복구
    ret
```

---

## 지역변수를 만들어보자

```
_add:
    push ebp
    mov ebp, esp

    ; 지역변수 생성 (int x, y)
    sub esp, 8  ; esp를 8바이트 만큼 민다

    mov eax, dword ptr[esp+8]
    add eax, dword ptr[esp+12]

    pop ebp
    ret
```

```
_main으로 돌아갈 주소
2
1
_asm_main으로 돌아갈 주소
ebp의 예전값 ( <- ebp )
X
X ( <- esp )
```

이러면 값을 넣을 경우 밀린 메모리에 값이 들어간다.

```
_add:
    push ebp
    mov ebp, esp

    ; 지역변수 생성 (int x, y)
    sub esp, 8  ; esp를 8바이트 만큼 민다

    mov dword ptr[ebp-4], 100
    mov dword ptr[ebp-8], 200

    mov eax, dword ptr[esp+8]
    add eax, dword ptr[esp+12]

    pop ebp
    ret
```

```
_main으로 돌아갈 주소
2
1
_asm_main으로 돌아갈 주소
ebp의 예전값 ( <- ebp )
100
200 ( <- esp )
```

지역변수의 파괴는?

```
_add:
    push ebp
    mov ebp, esp

    ; 지역변수 생성 (int x, y)
    sub esp, 8  ; esp를 8바이트 만큼 민다

    mov dword ptr[ebp-4], 100
    mov dword ptr[ebp-8], 200

    mov eax, dword ptr[esp+8]
    add eax, dword ptr[esp+12]

    ; 지역변수 파괴
    mov esp, ebp

    pop ebp
    ret
```

```
_main으로 돌아갈 주소
2
1
_asm_main으로 돌아갈 주소
ebp의 예전값 ( <- ebp, esp )
100 ( 스택에 남겨진 쓰레기 값이됨, 초기값을 선언하지 않을시 들어가는 그 쓰레기 값! )
200 
```

## 정리해보자.

어셈블리 함수의 일반적인 모양은 다음과 같다 할 수 있다.

```
_function:
    ; 함수의 prologue
    push ebp
    mov ebp, esp
    sub esp, 지역변수 크기
    ; ...
    ; 함수의 epilogue
    mov esp, ebp
    pop ebp
    ret
```

---

## 이전코드를 개선해보자.

```
_asm_main:
    push ebp
    mov ebp, esp

    push 2
    push 1
    call _add
    add esp, 8

    mov esp, ebp
    pop ebp
    ret

_add:
    push ebp
    mov ebp, esp

    mov dword ptr (-4)[ebp], 100     ; mov dword ptr[ebp-4], 100 와 동일
    mov dword ptr (-8)[ebp], 100

    move eax, dword ptr 8[ebp]
    move eax, dword ptr 12[ebp]

    mov esp, ebp
    pop ebp
    ret
```

_add 함수를 좀 더 간결화

```
x = -4
y = -8
a = 8
b = 12
_add:
    push ebp
    mov ebp, esp

    mov dword ptr x[ebp], 100
    mov dword ptr y[ebp], 100

    move eax, dword ptr a[ebp]
    move eax, dword ptr b[ebp]

    mov esp, ebp
    pop ebp
    ret
```

---

## cpp코드를 어셈블리로 예측해보자

```cpp
int add(int a, int b)
{
    int c = 0;
    int d = 0;
    c = a + b;
    return c;
}

int main()
{
    int n = add(1, 2);
}
```

```
.model flat

public _add
public _main

.code

_add:
    push ebp
    mov ebp, esp
    sub esp, 8

    ; int c = 0;
    ; int d = 0;
    mov dwrod ptr [ebp-4], 0
    mov dwrod ptr [ebp-8], 0

    ; c = a + b;
    mov eax, dword ptr [ebp+8]
    add eax, dword ptr [ebp+12]
    mov dword ptr[ebp-4], eax

    ; return c;
    mov eax, dword ptr [ebp-4]

    mov esp, ebp
    pop ebp
    ret

_main:
    push ebp
    mov ebp, esp
    ;sub esp, 4
    push ecx        ; 최적화를 위한 동작

    push 2
    push 1
    call _add
    add esp, 8

    mov dword ptr[ebp-4], eax

    ; return 0
    ; mov eax, 0
    xor eax, eax    ; 최적화

    mov esp, ebp
    pop ebp
    ret

end
```