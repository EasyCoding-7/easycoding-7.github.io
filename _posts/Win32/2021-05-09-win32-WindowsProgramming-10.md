---
layout: post
title:  "(Win32 : WindowsProgramming-10) 실행 파일 포맷(PE)"
summary: ""
author: win32
date: '2021-05-09 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-10/
---

## PE(Portable Executable Format)

![](/assets/img/posts/win32/WindowsProgramming-10-1.PNG){:class="img-fluid"}

크게 PE Header, PE Body로 나뉘며<br>
중요한 부분만 별도로 설명하자면<br>

* NT Header : 메모리 주소, 스택 크기 등을 보관
* Section .text : 함수 기계어 코드가 들어감
* Section .data : 전역 변수 초기값이 들어감
* Section .rdata : 상수 값이 들어감
* Section .idata : malloc, free등 dll정보
* Section .rsrc : 메뉴, 비트맵 등 실행파일이 사용하는 리소스

```cpp
#include <stdio.h>

char s[] = "abcdefg"; // Section .data

int main()
{
    void* p = malloc(sizeof(int)*100); // Section .idata
    free(p);

    printf("hello, world\n"); // Section .rdata
}
```

---

## 직접 빌드해서 실행파일을 확인해보자.

* 빌드 옵션
    * 32bit(x86)
    * Debug

```cpp
#include <stdio.h>

char s[] = "abcdefg";

int main()
{
    void* p = malloc(sizeof(int)*100);
    free(p);

    printf("hello, world\n");
}
```

빌드된 실행파일은 [PEView.exe](http://wjradburn.com/software/)를 통해서 확인

![](/assets/img/posts/win32/WindowsProgramming-10-2.PNG){:class="img-fluid"}

![](/assets/img/posts/win32/WindowsProgramming-10-3.PNG){:class="img-fluid"}

Image Base가 400000 인데 400000에 메모리가 올라가 있다는 말이다.

![](/assets/img/posts/win32/WindowsProgramming-10-4.PNG){:class="img-fluid"}

![](/assets/img/posts/win32/WindowsProgramming-10-5.PNG){:class="img-fluid"}

호출된 DLL 함수를 확인 가능하다.

참고) Debug, Release 차이점이 PEViewer를 통해 보면 확연하게 보이는데<br>
* 사용하지 않는 전역변수는 사라져있음
* DLL의 섹션이 rdata와 결합 되는 경우가 있음

모두 실행파일 크기를 줄이기 위해서 컴파일러가 최적화 해준 결과이다.

---

## 라이브러리 란?

```cpp
int add(int a, int b)
{
    return a + b;
}
```

add라는 함수가 너무 좋아서 많은 사람에게 배포하고 싶다.

문제점은?

1. 소스의 내용을 누구나 볼 수 있고 고칠수 있다.
2. 소스를 사용할때마다 컴파일 해야한다.

결론은 컴파일 된 상태로 배포하자.

* 정적 라이브러리(Static Link Library)
* 동적 라이브러리(Dynamic Link Library)

두 가지 방법이 있다.

---

## 차이점?

* 정적 라이브러리(Static Link Library)
    * .lib
    * 기계어에 해당 함수가 들어간다.
    * 장점 : 실행 파일의 배포가 쉽다.
    * 단점 : 매번 기계어가 포함되기에 메모리 사용량이 증가
    * 단점2 : 라이브러리가 업데이트될 경우 실행파일을 다시 빌드해야한다.
* 동적 라이브러리(Dynamic Link Library)
    * .dll
    * 실행파일에서 호출만 한다.
    * 장점 : 실행 파일의 메모리 사용량이 작다
    * 장점2 : 라이브러리 업데이트가 쉽다.
    * 단점 : dll을 항상 같이 배포해야한다.