---
layout: post
title:  "(C++ STL) chrono Example"
summary: ""
author: C++
date: '2021-06-10 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/stl/chrono/
---

```cpp
#include <iostream>
#include <chrono>
using namespace std;
using namespace chrono;

int main()
{
    hours h(1);
    minutes m(h);   // 60
    seconds s(h);   // 3600

    cout << m.count() << endl;
    cout << s.count() << endl;

    hours h2(s);    // error
    hours h2 = duration_cast<hours>(s); // ok

    using days = duration<int, ratio<3600*24, 1>>;  
    // 이런식으로 활용이 가능하다.
    days d(1);
    minutes m2(d);

    cout << m2.count() << endl; // 60 * 24
}
```

```cpp
#include <iostream>
#include <chrono>
using namespace std;
using namespace chrono;

int main()
{
    seconds s1(3);        // ok
    seconds s2 = 3;       // erorr
    seconds s3 = 3s;      // ok
    seconds s4 = 4min;      // ok

    cout << s4.count() << endl;
```

```cpp
#include <iostream>
#include <chrono>
using namespace std;
using namespace chrono;

void foo(seconds s) {}

int main()
{
    foo(3);     // error
    foo(3s);    // ok

    seconds s5 = 3min + 40s;        // ok
```

---

## 현재 시간구하기

```cpp
#include <iostream>
#include <chrono>
#include <string>
using namespace std;
using namespace chrono;

int main()
{
    system_clock::time_point tp = system_clock::now();
    // 1970년 1월 1일 0시 기준 얼마나 흘렀나 나온다.
    nanoseconds ns = tp.time_since_epoch();

    cout << ns.count() << endl;

    hours h = duration_cast<hours>(ns);
    cout << h.count() << endl;

    time_t t = system_clock::to_time_t(tp);
    string s = ctime(&t);
    cout << s << endl;
```