---
layout: post
title:  "(C++ STL) iterator : Basic"
summary: ""
author: C++
date: '2021-06-10 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/stl/iterator-basic/
---

```cpp
#include <iostream>
#include <list>
using namespace std;

int main()
{
    list<int> s = { 1, 2, 3, 4, 5 };

    // list<int>::iterator p = s.begin();
    // auto p = s.begin();
    auto p = begin(s);      // 이 구조가 제일 좋음 - C++11
    // 배열에도 먹히기 때문이다.

    int n = size(s);

    // 이거 주의
    auto p2 = end(s);
    *p2 = 10;       // error - end는 마지막 요소가 아니라 마지막 다음 요소임을 기억하자.
}
```

사용시 주의사항

```cpp
#include <iostream>
#include <list>
#include <vector>
using namespace std;

int main()
{
    vector<int> v = {1,2,3,4,5};

    auto p = begin(v);

    v.resize(100);          // 버퍼 재할당

    cout << *p << endl;     // error - 버퍼가 재할당되며 무효화 된다.
}
```

---

## range 활용

```cpp
#include <iostream>
#include <list>
using namespace std;

int main()
{
    list<int> s1;

    if(s1.empty()) {}       // 요소가 비어있는지 확인
}
```

```cpp
#include <iostream>
#include <list>
#include <algorithm>
using namespace std;

int main()
{
    int x[5] = {1,2,3,4,5};
    int y[5] = {0,0,0,0,0};

    list<int> s2 = {0,0,0,0,0};

    copy(x, x+5, y);
    copy(x, x+5, begin(s2));

    for( auto& h : y)
        cout << h << ", ";

    for( auto& h : s2)
        cout << h << ", ";
}
```
