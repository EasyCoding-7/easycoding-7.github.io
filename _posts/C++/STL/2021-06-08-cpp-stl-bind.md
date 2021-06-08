---
layout: post
title:  "(C++ STL) bind"
summary: ""
author: C++
date: '2021-06-08 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/stl/bind/
---

```cpp
#include <iostream>
#include <functional>
using namespace std;
using namespace std::placeholders;

void f1(int a, int b, int c) { printf("f1 : %d, %d, %d\n", a,b,c); }
void f2(int& a) { a = 20; }

int main()
{
    bind(&f1, 1, 2, 3)();       // f1(1,2,3)으로 고정된다.
    bind(&f1, 1, 2, _1)(10);    // f1(1,2,10)으로 고정된다.
    bind(&f1, 1, _2, _1)(10, 20);// f1(1,20,10)으로 고정된다.
}
```

```cpp
int n = 0;
bind(&f2, n)();     // f2(n)

cout << n << endl;  // 20? or 0? -> 0이 나온다.

reference_wrapper<int> r(n);    // 20이 나온다.

bind(&f2, ref(n))();  // reference_wrapper를 매번치기 힘드니.
```

```cpp
#include <iostream>
#include <functional>
using namespace std;
using namespace std::placeholders;

class Test
{
public:
    int data = 0;
    void f(int a, int b)    // == void f(Test* this, int a, int b)
    {
        data = a;
        printf("f : %d, %d\n", a, b);
    }
};

int main()
{
    Test t;
    bind(&Test::f, &t, 1, 2)(); // t.f(1,2)
    // bind(&Test::f, ref(t), 1, 2)(); // 동일표현

    bind(&Test:data, &t)() = 10;    // t.data = 10

    cout << t.data << endl;
}
```