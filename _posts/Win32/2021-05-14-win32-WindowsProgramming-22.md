---
layout: post
title:  "(Win32 : WindowsProgramming-22) Virtual Address Space"
summary: ""
author: win32
date: '2021-05-14 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-22/
---

## 가상 메모리

* 물리 메모리 : RAM
* 물리 메모리만으로 프로그램을 돌리기엔 부족하다
* 잠깐 보관할 공간(paging file)을 HDD/SDD에 할당하자
* 이런 기법을 이용할 시 훨씬더 많은 메모리 공간을 사용가능 (가상 메모리)

---

## 페이지(PAGE)

* 운영체제가 메모리를 관리 할 때 사용하는 최소 단위
* x64, x84 : 4K(4096) / IA-64 : 8K(8192)

```

   Physical Storage
   |               |            Process A
   |   PAGE 10     |    <----- |  0x1234 |
   |               |           |---------|            Process B
   |   PAGE 20     |    <--------------------------- |  0x1234 |
   |---------------|                                 |---------|

```

* 모든 프로세스는 가상주소를 사용
    * 다른 프로세스가 사용하는 메모리에 접근 불가
    * 같은 주소라도 프로세스가 다른 경우 다른 물리 공간을 사용

```cpp
#include <stdio.h>
#include <Windows.h>

int main()
{
    //# PAGE 크기 구하기
    SYSTEM_INFO si = {0};
    GetSystemInfo(&si);
    printf("PAGE SIZE : %d\n", si.dwPageSize);      // 4K단위로 할당될 것

    //# 물리메모리 크기, Page File 크기 얻기
    MEMORYSTATUSEX mse = {0};
    mse.dwLength = sizeof(mse);

    GlobalMemoryStatusEx(&mse);

    printf("Total Phy Mem : %lld\n", mse.ullTotalPhys);     // 설치된 물리메모리 크기
    printf("Available Phy Mem : %lld\n", mse.ullAvailPhys); // 할당 가능한 물리 메모리 크기
    printf("Total Page File : %lld\n", mse.ullTotalPageFile);   // Paging File의 최대 크기
}
```

---

## 가상 주소 공간 파티션

```

/* 프로세스 주소공간 */

   0xFFFF FFFF
|               |
|    OS 코드    |   << Kernel Mode Partition >>
| Device Driver |   디바이스 드라이버 코드에서만 접근가능
| Kernel Object |   일반 어플리케이션에서 접근 안됨.
|               |
|  0x8000 0000  |
|               |
|               |   << User Mode Partition >>
|               |   실행파일, DLL, Stack, Heap 등이 매핑되는 공간
|               |
|               |
   0x0000 0000

```

* User Mode Partition은 모든 프로세스가 독립적으로 사용한다.
* Kernel Mode Partition 은 포든 프로세스에서 같은 피지컬 메모리공간을 사용한다.

```cpp
#include <stdio.h>
#include <Windows.h>

int main()
{
    SYSTEM_INFO si = {0};
    GetSystemInfo(&si);
    printf("PAGE SIZE : %d\n", si.dwPageSize);
    printf("Max Addr : %p\n", si.lpMaximumApplicationAddress);
    printf("Max Addr : %p\n", si.lpMinimumApplicationAddress);


    MEMORYSTATUSEX mse = {0};
    mse.dwLength = sizeof(mse);

    GlobalMemoryStatusEx(&mse);

    printf("Total Phy Mem : %lld\n", mse.ullTotalPhys);
    printf("Available Phy Mem : %lld\n", mse.ullAvailPhys);
    printf("Total Page File : %lld\n", mse.ullTotalPageFile);
}
```

---

```

/* 프로세스 주소공간 */

       0xFFFF FFFF
|  Kernel Mode Partition  |
|       0x8000 0000       |
|         XXX.dll         |
|         .exe            | <- 0x0040 0000 
|                         |
|                         |
|                         |
       0x0000 0000

```

* 특별한 설정을 하지 않는다면 실행파일은 `0x0040 0000`에 매핑되어 있음.
    * 단, 현재는 보안상의 문제로 무조건 `0x0040 0000`에 올라가진 않는다
* .dll, .exe를 모듈이라하며 `GetModuleHandle()` 함수를 호출시 모듈의 시작주소를 받을 수 있다.

```cpp
#include <Windows.h>
#include <stdio.h>

void print_module_address(const char* name)
{
    HANDLE hMod = GetModuleHandleA(name);

    if(name == 0) name = ".exe";
    printf("%15s : %p\n", name, hMod);
}

int main()
{
    HANDLE h = GetModuleHandleA("user32.dll");

    print_module_address(0);
    print_module_address("kernel32.dll");
    print_module_address("user32.dll");
    print_module_address("gdi32.dll");
    print_module_address("ntdll.dll");
}
```

---

## 메모리 보호 속성

* [속성목록](https://docs.microsoft.com/en-us/windows/win32/memory/memory-protection-constants)

* 보호 속성 조사 : `VirtualQuery()`
* 보호 속성 변경 : `VirtualProtect()`

```cpp
int main()
{
    int y = 0;

    MEMORY_BASIC_INFORMATION mbi = {0};
    VirtualQuery(&y, &mbi, sizeof(mbi));    // 지역변수(Stack)의 보호 속성을 보자.
    printf("%p : %d\n", &y, mbi.Protect);   // PAGE_READWRITE(말 그대로 읽고 쓰기 가능)
}
```

모든 변수로 확장해 보자.

* 함수주소(.Text) : PAGE_EXECUTE_READ
* 전역변수 주소(.data) : PAGE_READWRITE
* 지역변수 주소(Stack) : PAGE_READWRITE

```

* 보호속성의 적용은 PAGE단위로 적용이 된다.
* Section마다 보호속성이 다르다.

   /* 실행파일 PE 포맷 */               /* 프로세스 가상주소 */

  |---------------------|              |---------------------|
  |                     |              |                     |
  |     PE Header       |              |     PE Header       |
  |                     |              |                     |
  |---------------------|              |---------------------|
  |                     |              |                     |
  |    Section .Text    |              |      .text          |
  |       (함수)        |              |   PAGE_EXECUTE_READ |
  |                     |              |---------------------|
  |---------------------|              |                     |
  |                     |              |      .data          |
  |    Section .data    |              |   PAGE_READWRITE    |
  |  (초기화된 전역변수) |              |---------------------|
  |                     |              |                     |
  |---------------------|              |       ...           |
  |                     |              |                     |
  |                     |              |---------------------|
  |                     |              |     Stack           |
                                            PAGE_READWRITE
            ...

 지역변수는 별도의 Stack에 포함된다.

```

## 보호 속성 변경해 보기

```cpp
#include <stdio.h>
#include <Windows.h>

int main()
{
    char s1[] = "hello";
    char* s2 = "hello";

    *s1 = 'A';  // okay
    *s2 = 'A';  // runtime error
}
```

왜 에러가 발생?

```cpp
char s1[] = "hello";        // Stack에 메모리 할당 (PAGE_READWRITE)
char* s2 = "hello";         // Stack에 포인터 변수만 할당, 그럼 실제 문자열은? (Section .rdata에 할당됨)
```

```

   /* 실행파일 PE 포맷 */
  |---------------------|
  |                     |
  |     PE Header       |
  |                     |
  |---------------------|
  |                     |
  |    Section .Text    |
  |       (함수)        |
  |                     |
  |---------------------|
  |                     |
  |    Section .data    |
  |  (초기화된 전역변수) |
  |                     |
  |---------------------|
  |                     |
  |    Section .rdata   |
  |    PAGE_READONLY    |
  |                     |
            ...

```

```cpp
#include <stdio.h>
#include <Windows.h>

int main()
{
    char s1[] = "hello";
    char* s2 = "hello";

    DWORD old = 0;
    VirtualProtect(s2, strlen(s2)+1, PAGE_READWRITE, &old);
    // Section .data를 PAGE_READWRITE로 바꿔달라

    *s1 = 'A';
    *s2 = 'A';
}
```