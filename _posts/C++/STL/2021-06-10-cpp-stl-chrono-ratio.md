---
layout: post
title:  "(C++ STL) chrono : ratio(분수사용하기)"
summary: ""
author: C++
date: '2021-06-10 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/stl/chrono-ratio/
---

```cpp
// 내부구현은 이렇고
template<intmax_t _Nx, intmax_t _Dx = 1>
struct ratio
{
    static constexpr intmax_t num = _Nx;
    static constexpr intmax_t den = _Dx;

    typedef ratio<num, den> type;
};
```

```cpp
// 실사용은 이렇게 하자.
#include <iostream>
#include <ratio>
using namespace std;

int main()
{
    ratio<2, 4> r1;     // 2/4 = 1/2

    cout << r1.num << endl;     // 1
    cout << r1.den << endl;     // 2

    cout << ratio<2, 4>::num << endl;   // 1
    cout << ratio<2, 4>::den << endl;   // 2

    cout << sizeof(r1) << endl;     // empty (컴파일 시간에 출력하기에 메모리를 따로 갖지 않는다.)
}
```

간단한 연산도 가능하다

```cpp
#include <iostream>
#include <ratio>
using namespace std;

int main()
{
    ratio_add<ratio<1,4>, ratio<2,4>> r2;

    cout << r2.num << endl;
    cout << r2.den << endl;
}
```

단위 연산도 제공

```cpp
milli m;
kilo k;

cout << k.num << endl;      // 1000
cout << k.den << endl;      // 1
```