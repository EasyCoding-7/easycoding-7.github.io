---
layout: post
title:  "(Win32 : WindowsProgramming-24) Heap Memory"
summary: ""
author: win32
date: '2021-05-14 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-24/
---

```cpp
#include <stdio.h>
#include <Windows.h>

int main()
{
    void* p = VirtualAlloc(
                0,      // 메모리 주소
                100,    // 메모리 크기(4096)
                MEM_RESERVE | MEM_COMMIT,
                PAGE_READWRITE);

    VirtualFree(p, 0, MEM_RELEASE);
}
```

만약 진짜로 100Mb만 필요했다면??<br>
일단은 4096Mb를 할당후 할당된 4096안에서 나눠서 쓴다<br>
우선 100을 쓰고 나중에 또 필요하면 거기서 또 쓰고 이런식

이런 방법이 Windows API 자체적으로 구현이 되어있다.

---

Heap은 기본적으로 프로세스가 생성될 시 1Mb를 할당해 주는데 이 기본 Heap을 사용할 경우 -> `GetProcessHeap() -> HeapAlloc() -> HeapFree()` 의 절차<br>
사용자가 추가로 Heap을 만드는 경우 -> 뒤에서 알려줌.

```cpp
#include <stdio.h>
#include <Windows.h>

int main()
{
    HANDLE heap = GetProcessHeap();
    int* p1 = (int*)HeapAlloc(heap,
                            HEAP_ZERO_MEMORY    // 0으로 메모리 초기화
                            sizeof(int) * 10);

    printf("%p\n", p1);

    p1[0] = 10;
    p1[1] = 20;

    HeapFree(heap, 0, p1);
}
```

* 큰 블록(1M이상)의 메모리의 경우
    * `VirtualAlloc()`을 사용하는 것을 권장
* 작은 크기의 메모리 블럭
    * Heap을 사용하는 것을 권장
* malloc(new) Vs HeapAlloc의 차이점?
    * 뒤에서 설명

---

힙의 크기확인/확장

```cpp
#include <stdio.h>
#include <Windows.h>

int main()
{
    HANDLE heap = GetProcessHeap();
    int* p1 = (int*)HeapAlloc(heap,
                            HEAP_ZERO_MEMORY
                            40);

    int sz = HeapSize(heap, 0, p1); // 40
    
    HeapReAlloc(heap, 0, p1, 30);   // 40 -> 30
    int* p2 = HeapReAlloc(heap, 0, p1, 60);   // 30 -> 60
    // 리턴으로 메모리 주소를 받는 것은 메모리 공간을 이전하여 확장할 수 있기 때문이다.

    HeapFree(heap, 0, p1);
}
```

---

## 추가 힙 생성

* `HeapCreate() -> HeapDestroy()`
* 그런데 왜 기본 힙을 사용하지 않을까?

힙을 하나만 사용했을때 단점을 우선 알아야 한다.

Linked list 인 Node 1, 2를 Heap 에 할당했다 가정해보자.<br>
Heap 에는 아래와 같이 할당 될텐데

```
<< Heap 1 >>
| Node1 | Node 2 | Node 1 | Node 2 |
```

문제점

* Node1에서 메모리접근 실수로 Node2에 문제를 발생시킬수 있음
* Node1의 메모리를 해지해도 메모리의 이득이 없다(메모리 단편화(fragment) 현상)

```
<< Heap 1 >>
| Node 1 | Node 1 | 

<< Heap 2 >>
| Node 2 | Node 2 |
```

* 자료구조별로 다른 힙을 사용하면
    * 안정성이 좋고, 메모리 단편화 현상이 해결된다.
    * 메모리 해지시 Heap을 해지해버리면 된다.
    * 기본Heap의 경우 다양한 API함수가 내부적으로 메모리가 필요할때 사용되기에 사용이 잦다
    * `HeapAlloc()` Serialize(동기화)되는데 다양한 Thread에서 접근가능성이 있기때문이다. 이부분도 성능에 문제를 발생시킬 수 있다.
    * 내가 만든 Heap은 Serialize를 안시킬수 있는 옵션이 별도로 있어 성능향상을 볼 수 있다.

```cpp
#include <stdio.h>
#include <Windows.h>

int main()
{
    HANDLE heap1 = GetProcessHeap();
    HANDLE heap2 = HeapCreate(HEAP_NO_SERIALIZE,    // 동기화는 하지마라
                                4096,               // 확정크기
                                0);                 // 최대 크기(0 : 무제한)

    void* p1 = HeapAlloc(heap1, HEAP_ZERO_MEMORY, 100);
    void* p2 = HeapAlloc(heap2, HEAP_ZERO_MEMORY, 100);

    printf("%p\n", p1);
    printf("%p\n", p2);

    HeapFree(heap1, 0, p1);
    HeapFree(heap2, 0, p2);

    // 기본힙은 프로세스 종료시에 자동으로 HeapDestroy호출됨.
    HeapDestroy(heap2);
}
```

생성된 힙 순회하기

```cpp
#include <stdio.h>
#include <Windows.h>

int main()
{
    HANDLE heap1 = GetProcessHeap();
    HANDLE heap2 = HeapCreate(0, 4096, 0);

    HANDLE handle[256];
    int cnt = GetPRocessHeaps(256, handle);


    for(int i = 0; i < cnt; i++)
    {
        printf("%d : %x\n", i, handle[i]);
    }

    HeapDestroy(heap2);
}
```

---

## CRT vs API

* `malloc / new` 도 결국 `HeapAlloc() or VirtualAlloc()`을 호출하게 된다.

* Windows OS만을 위해 만들어진 프로그램이다 -> `HeapAlloc()`쓰는 것을 추천
* Linux등 다른 OS도 지원해야 한다 -> `malloc / new` 사용

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <Windows.h>

int main()
{
    HANDLE h1 = GetProcessHeap();

    void* p1 = HeapAlloc(h1, 0, 100)
    void* p2 = malloc(100);
    void* p3 = new char[100];

    // 세 주소다 다 비슷한곳에 할당됨(결국 모두 HeapAlloc() 쓴다는 말)
    printf("HeapAlloc : %p\n", p1);
    printf("malloc : %p\n", p2);
    printf("new : %p\n", p3);

    delete[] p3;
    free(p2);
    HeapFree(h1, 0, p1);
}
```