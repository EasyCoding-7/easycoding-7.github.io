---
layout: post
title:  "(C++ : Move Semantics-4) move 어디쓰나?"
summary: ""
author: C++
date: '2021-05-21 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/move-semantics/4/
---

```cpp
class Test
{
publid:
  Test() {}
  ~Test() {}
  Test(const Test& t) { cout << "Copy" << endl; }
  Test(Test&& t)      { cout << "Move" << endl; }
  
  Test& operator=(const Test& t)
  {
    cout << "Copy=" << endl;
    return *this;
  }
  Test& operator=(Test&& t)
  {
    cout << "Move=" << endl;
    return *this;
  }
};

template<typename T> void Swap(T& x, T& y)
{
  Test temp = x;    // 복사 생성자 호출
  x = y;            // 복사 대입 호출
  y = temp;         // 복사가 너무 잦군... -> 성능저하의 요인이 된다.
}

int main()
{
  Test t1, t2;
  Swap(t1, t2);
}
```

```cpp
template<typename T> void Swap(T& x, T& y)
{
  Test temp = move(x);
  x = move(y);
  y = move(temp);
}
```

```cpp
// 이럴줄 알고 STL이 준비했다.
#include <algorithm>

int main()
{
  Test t1, t2;
  swap(t1, t2);   // STL에서 기본으로 제공해주는 함수. -> move를 이용해 복사한다.
}
```

---

```cpp
int main()
{
  Test* p1 = new Test[2];
  // 버퍼를 늘리고 싶다 2 -> 4
  Test* p2 = new Test[4];
  
  for(int i = 0; i < 2; i++)
    p2[i] = p1[i];          // copy 대입! -> 역시 성능저하가 발생
    p2[i] = move(p1[i]);    // move 대입 -> 성능 향상을 본다.
}
```

```cpp
vector<Test> v(2);
v.resize(4);      // copy 대입이 호출되는데 강제로 move 대입을 호출을 원한다면
// 아래와 같이 noexcept 선언

class Test
{
publid:
  Test() {}
  ~Test() {}
  Test(const Test& t) { cout << "Copy" << endl; }
  Test(Test&& t) noexcept { cout << "Move" << endl; }
  
  Test& operator=(const Test& t)
  {
    cout << "Copy=" << endl;
    return *this;
  }
  Test& operator=(Test&& t) noexcept
  {
    cout << "Move=" << endl;
    return *this;
  }
};
```

```cpp
#include <iostream>
#include <vector>
#include <type_traits>
#include "Test.h"
using namespace std;

int main()
{
  Test* p1 = new Test[2];
  Test* p2 = new Test[4];
  
  for(int i = 0; i < 2; i++)
    p2[i] = move(p1[i]);
    // move를 진행 중 예외가 발생하면 큰 문제가 발생할 수 있음 -> STL에서 noexcept를 강제하는 이유
}
```

```cpp
Test t1;
Test t2 = t1;   // copy
Test t3 = move(t2); // move

bool b = is_nothrow_move_constructible<Test>::value;
// Test의 move생성자에 예외가 있을 시 true

cout << b << endl;    // true

Test t4 = move_if_noexcept(t1);   // 예외가 없다면 move 생성자 호출
```

* 버퍼를 복사 할 시 무조건 move쓴다고 move로 동작하지는 않는다.
* 예외를 던지는 지 확인하고 move생성자에 noexcept을 붙이는 것을 잊지말자