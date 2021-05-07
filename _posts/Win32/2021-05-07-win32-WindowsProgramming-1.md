---
layout: post
title:  "(Win32 : WindowsProgramming-1) Project Setting"
summary: ""
author: win32
date: '2021-05-07 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-1/
---

* [Get Code](https://github.com/EasyCoding-7/Win32Example.git)

---

## VC++ 확장 문법 제거

```cpp
struct Point
{
    int x, y;
};

int main()
{
    Point& p = Point();     // VS에서 에러없이 동작.
    // 이걸 에러로 처리하고 싶다면?
}
```

![](/assets/img/posts/win32/WindowsProgramming-1-1.PNG){:class="img-fluid"}

`/Za`옵션을 넣는다가 핵심

---

## CL 컴파일러 사용

* `cl.exe` : MS에서 제공하는 C/C++컴파일러
* x64 Native Tools Command Prompt for VS 2019를 통해서 실행가능

![](/assets/img/posts/win32/WindowsProgramming-1-2.PNG){:class="img-fluid"}

cl컴파일러는 다양한기능이 있는데 `/EP`를 옵션으로 넣을경우 전처리기만 처리하고 코드결과를 출력해 달라는 명령이다.

```cpp
#define MAX 10
#define SQUARE(a) ((a)*(a))

int main()
{
    int arr[MAX] = {0};
    int n = SQUARE(3);
}
```

![](/assets/img/posts/win32/WindowsProgramming-1-3.PNG){:class="img-fluid"}

참고로 `cl /help`를 사용시 cl컴파일러의 다양한 옵션을 확인할 수 있다.