---
layout: post
title:  "(Win32 : WindowsProgramming-7) Window API 개념"
summary: ""
author: win32
date: '2021-05-08 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-7/
---

## Win32 API 개념

* Windows OS에서 실행되는 프로그램을 개발하기 위한 SDK

```cpp
#include <stdio.h>
#include <Windows.h>    // for Win32 API

int main()
{
    printf("Hello, C\n");

    MessageBoxA(0, "Hello", "API", MB_OK);
}
```

---

## API함수의 호출규약

* C 표준 함수 : `__cdecl`
* Windows API 함수 : `__stdcall`

---

## 동일한 함수가 3가지 형태로 제공되는 형태가 있다.

```cpp
MessageBoxA(0, "Hello", "API", MB_OK);
MessageBoxW(0, L"Hello", L"API", MB_OK);
MessageBox (0, _T("Hello"), _T("API"), MB_OK);
```

자세한 설명은 UNICODE를 설명후 진행

---

## UNICODE

```cpp
#include <stdio.h>
#include <string.h>

int main()
{
    char s[] = "abcd가나다라";

    printf("%d\n", sizeof(s));  // 13(마지막에 null 포함)
    printf("%d\n", strlen(s));  // 12
}
```

* SBCS(Single Byte Character Set) : 한 바이트를 사용해서 문자를 표현, 256개 밖에 사용불가 영어이외의 언어를 표현하지 못함.
* MBCS(Multi Byte Character Set) : 하나의 문자를 표현하기 위해 다양한 바이트 수를 사용, 윈도우OS에서 영어는 1바이트, 한글은 2바이트 (보통 2바이트 정로를 사용하기에 DBCS(Double Bytes Character Set)이라고도 한다.)

### DBCS문제점

```cpp
#include <stdio.h>
#include <string.h>
#include <string.h>

int main()
{
    char s[] = "abcd가나다라";  // DBCS

    char* p = s;

    while(*p != 0)
    {
        printf("%s\n", p);
        p = p + 1;
        /*
        영어는 1바이트만 추가해도 다음 문자열이 나오는데,
        한글은 2바이트를 추가해야 다음 문자열을 출력할 수 있다.
        */
    }
}
```

해결

```cpp
while(*p != 0)
{
    printf("%s\n", p);
    if(IsDBCSLeadByte(*p))
        p = p + 2;
    else
        p = p + 1;
}
```

```cpp
while(*p != 0)
{
    printf("%s\n", p);
    p = CharNextA(p);
}
```

* DBCS 문제점
    * 하나의 문자를 1, 2bytes로 표현하기에 프로그램 작성시 문제점이 있다.
    * 나라별로 표쥰이 다르기에 호환성 문제가 있다.

결론 : 전세계 대부분의 문자를 동시에 표현할 수 있는 새로운 문자 집합이 필요하다 -> Unicode

---

## Unicode

* UTF-8 : 가변 길이로 인코딩, 1,2,3,4 바이트 모두 사용 언어에 따라 다름(영어 1, 한글 3바이트 사용)
    * 아무래도 영어를 많이 사용하는 환경에서 사용
* UTF-16 : 가변 길이로 인코딩, 대부분 문자 2바이트 특정문자 4바이트(영어 2, 한글 2바이트 사용)
    * 한글과 영어가 비슷하게 사용되는 환경에서 사용
* UTF-32 : 모든 문자를 4바이트로 인코딩, 단 메모리 사용량 증가
    * 리눅스에서 사용

* Windows OS는 UTF-16을 사용
    * Linux의 경우 UTF-32임 참고

```cpp
#include <stdio.h>
#include <string.h>

int main()
{
    char s1[] = "abcd가나다라"; // DBCS
    wchar_t s2[] = L"abcd가나다라";

    // L"" -> 문자열을 유니코드로 만들어 달라 -> Windows의 경우 UTF-16이고, 2byte

    /*
    whar_t :
        Windows : 2byte
        Linux : 4byte, 리눅스의 경우 UTF-32를 쓰니 그렇겠지?
    */

    printf("%d\n", sizeof(s2));         // 18(null포함)
    printf("%d\n", strlen((char*)s2));  // 1 -> why?
    // 제일 처음 나오는 문자 a는 아스키코드로 65이고 2바이트로 표현시 65, 00
    // little endian 표현으로 65, 00로 표현하게 되는데
    // strlen은 00(null)을 만날시 마지막 문자로 인식해서 길이를 리턴하게 된다.

    // 유니코드의 문자열 개수를 세는 새로운 len함수가 필요하다
}
```

```cpp
printf("%d\n", wcslen(s2));

// 출력 또한 유니코드용이 있음
wprintf(L"AA\n");

MessageBoxA(0, "A", "B", MB_OK);
MessageBoxW(0, L"A", L"B", MB_OK);
```

---

## 유니코드 매크로

```cpp
#include <tchar.h>

int main()
{
    TCHAR s[] = _T("abcd가나다라");

    // 유니코드일때는 유니코드로 쓰게해 달라

    _tprintf(_T("%d\n"), _tsclen(s));

    MessageBox(0, _T("A"), _T("B"), MB_OK);
}
```

(이건 참고)

main함수도 매개변수로 `char**` 유니코드가 아닌 문자열을 받지 않나?<br>
main도 종류별로 나눠야 한다.

-> `main, wmain, _tmain`

**물론 Unicode를 무조건 써야 한다는 것은 아니다. 다만 권장사항일 뿐!**

---

## GetLastError()

* errno
    * 에러의 원인을 담은 변수
    * 구현에 따라 Thread-safe할 수 있고 아닐 수 있다는 단점이 있다.

* GetLastError()
    * Window API함수 호출이 실패 한 경우 실패의 원인을 담은 번호를 반환
    * Thread-safe하다

```cpp
#define _CRT_SECURE_NO_WARNING
#include <stdio.h>
#include <Windows.h>

int main()
{
    FILE* f = fopen("a.txt", "rt"); // 없는 파일임

    if (f == 0)
        printf("실패 : %d\n", errno); // 2

    HWND hwnd = CreateWindow(0, 0, 0, 0, 0,
                             0, 0, 0, 0, 0, 0); // 의도적으로 Invalid한 값을 넘긺

    if(hwndd == 0)
        printf("실패 : %d, %d\n", errno, GetLastError()); // 2, 86
    // errno이 잘못된 값을 리턴함을 알수있다.
}
```

참고로 도구 -> 오류조회 에서 에러타입에 대한 설명을 볼 수 있음.

![](/assets/img/posts/win32/WindowsProgramming-7-1.PNG){:class="img-fluid"}