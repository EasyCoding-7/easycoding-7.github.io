---
layout: post
title:  "(C++ STL) chrono : duration(단위표현법)"
summary: ""
author: C++
date: '2021-06-10 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/stl/chrono-duration/
---

```cpp
#include <iostream>
#include <chrono>
using namespace std;
using namespace chrono;

int main()
{
    double distance = 3;        // 단위를 나타낼 수 없다

    /* ******************* */

    duration<double, ratio<1,1>> d1 = 3;        // 3m
    duration<double, ratio<1,1000>> d2(d1);     // milli 3000

    cout << d2.count() << endl; // 3000

    duration<double, ratio<1000, 1>> d3(d1);    // km

    cout << d3.count() << endl;
}
```

```cpp
// 자주사용되는 부분은 정의됨
duration<double, ratio<1,1>> d1 = 3;    // 3m
duration<double, milli> d2(d1);         // milli 3000

cout << d2.count() << endl;             // 3000

duration<double, kilo> d3(d1);          // km

cout << d3.count() << endl;
```

```cpp
using MilliMeter = duration<int, milli>;
using KiloMeter = duration<int, kilo>;
using Meter = duration<int, ratio<1,1>>;

Meter m(600);       // 600m
MilliMeter mm(m);   // 600000
KiloMeter km(m);    // 0.6?? -> int라서 불가능! - error

KiloMeter km = duration_cast<KiloMeter>(m);     // 0 - 버림으로 받는다
// floor, ceil, round, abs등을 C++17부터 제공한다.
```