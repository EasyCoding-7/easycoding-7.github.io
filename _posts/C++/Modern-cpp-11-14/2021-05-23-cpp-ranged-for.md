---
layout: post
title:  "(Modern C++ : 11~14) Ranged for"
summary: ""
author: C++
date: '2021-05-23 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/modern-cpp-11-14/ranged-for/
---

```cpp
#include <iostream>
#include <list>
using namespace std;

int main()
{
    // int x[10] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };       // ok
    list<int> s = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };        // ok

    for(auto& n : x)
        cout << n << endl;

    /*
    for(int i = 0; i < 10; i++)
        cout << x[i] << endl;
    */
}
```

```cpp
int main()
{
    // int x[10] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };       // ok
    list<int> s = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };        // ok

    for(auto& n : s)
        cout << n << endl;

    /* 위와 동일한 표현이다.
    for(auto p = being(s); p != end(s); ++p)
        auto& n = *p;
        cout << n << endl;
    */
}
```