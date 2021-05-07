---
layout: post
title:  "(Win32 : WindowsProgramming-2) 어셈블리와 MASM"
summary: ""
author: win32
date: '2021-05-07 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-2/
---

## 테스트 환경

* Intel CPU
* Windows10
* 32bits

---

```cpp
// inline_asm1.c
#include <stdio.h>

int Add(int a, int b)
{
    int c = a + b;
    return c;
}

int main()
{
    int n = 0;
    // 스택에 매개변수 1, 2를 넣을 것
    n = Add(1, 2);
    // 반환 값은 EAX 레지스터에 담겨서 돌아 올 것
    printf("result : %d\n", n);
}
```

확인해 보자.

VS의 인라인 어셈블리를 사용하면 되고, 주의할 점은 32bits만 지원한다.

```cpp
// Example
__asm
{
    mov n, eax
}
```

```cpp
// inline_asm1.c
#include <stdio.h>

int Add(int a, int b)
{
    int c = a + b;
    return c;
}

int main()
{
    int n = 0;
    Add(1, 2);
    
    __asm
    {
        mov n, eax
    }

    printf("result : %d\n", n);
}
```

```
result : 3
```

```cpp
// inline_asm1.c
#include <stdio.h>

int Add(int a, int b)
{
    int c = a + b;

    __asm
    {
        mov eax, c  // return c
    }
}

int main()
{
    int n = 0;
    Add(1, 2);
    
    printf("result : %d\n", n);
}
```

---

## 함수 호출을 인라인어셈블로 처리

함수의 매개변수는 스택에 넣는다.

```cpp
// inline_asm1.c
#include <stdio.h>

int Add(int a, int b)
{
    return a + b;
}

int main()
{
    int n = 0;
    //n = Add(1, 2);

    __asm
    {
        push    2
        push    1
        call    Add
        add     esp, 8  // 할당된 스택을 제거한다.(일단은 받아들이자.)

        mov     n, eax
    }
    
    printf("result : %d\n", n);
}
```

---

## 어셈블리 파일로 만들어서 빌드해보기

```cpp
#include <stdio.h>

int asm_main();     // 어셈블리 파일로 만들예정

int main()
{
    int n = asm_main();
    printf("result : %d\n", n);
}
```

```
; asm1.asm  ; 어셈블리는 세미콜론이 주석

.model flat

public _asm_main

.code
_asm_main:
    mov     eax, 100        ; return 100과 동일
    ret

end
```

빌드방법?

* command line
* 사용자 지정빌드 명령 추가
* 빌드 종속성 추가

---

### command line

```
cl main.c /c
ml asm1.asm /c
link main.obj asm1.obj
```

* `/c` : 링크없이 컴파일만 해달라

---

### 사용자 지정빌드 명령 추가

![](/assets/img/posts/win32/WindowsProgramming-2-1.PNG){:class="img-fluid"}

![](/assets/img/posts/win32/WindowsProgramming-2-2.PNG){:class="img-fluid"}

모든 asm파일에 이렇게 해야하나? 100개, 1000개면?? -> 3번째 방식으로 해결

---

### 빌드 종속성 추가

![](/assets/img/posts/win32/WindowsProgramming-2-3.PNG){:class="img-fluid"}

![](/assets/img/posts/win32/WindowsProgramming-2-4.PNG){:class="img-fluid"}