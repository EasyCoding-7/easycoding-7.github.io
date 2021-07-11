---
layout: post
title:  "(C++ STL) function"
summary: ""
author: C++
date: '2021-07-11 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/stl/function/
---

## bind

우선 bind부터 알아야 한다.

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
void f2(int& a) { a = 20; }

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

---

## Function

```cpp
#include <iostream>
#include <functional>
using namespace std;
using namespace std::placeholders;

void f1(int a) { printf("f1 : %d\n", a); }
void f2(int a, int b, int c) { printf("f2 : %d, %d, %d\n", a, b, c); }

int main()
{
    void(*f)(int) = &f1;        // ok
    void(*f)(int) = &f2;        // error

    // 함수 포인터는 signature가 동일해야한다

    function<void(int)> f;      // 리턴이 void이고 입력이 int면 다된다.
    
    f = &f1;                    // ok
    f = bind(&f2, 1,2,_1)       // ok
    f(10);                      // f2(1,2,10);

    bind(&f2, 1,2,_1)(10);      // 는 그냥 호출임을 기억
}
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
    void f(int a, int b)
    {
        data = a;
        printf("f : %d, %d\n", a, b);
    }
};

int main()
{
    Test t;

    function<void(int)> f1;
    f1 = bind(&Test::f, &t, _1, 20);

    // 일반 함수 모양의 function
    f1(10); // t.f1(10, 20);

    // 객체를 function의 인자로 받는 방법
    function<void(Test*, int)> f2;
    f2 = bind(&Test::f, _1, _2, 20);
    f2(&t, 100);        // t.f(100, 20)

    function<void(Test, int)> f3;
    f3 = bind(&Test::f, _1, _2, 20);
    f3(&t, 200);        // t.f(200, 20)

    function<void(Test&, int)> f4;
    f4 = bind(&Test::f, _1, _2, 20);
    f4(&t, 300);        // t.f(200, 20)

    cout << t.data << endl;
}
```

```cpp
#include <iostream>
#include <functional>
using namespace std;
using namespace std::placeholders;

class Test
{
public:
    int data = 10;
};

int main()
{
    Test t1, t2;

    function<int()> f1;
    f1 = bind(&Test::data, &t1);    // t1.data

    cout << f1() << endl;           // t1.data get

    f1() = 20;                      // error - functiond이 값으로 리턴하기에

    function<int&()> f2;
    f2() = 20;                      // ok

    function<int&(Test*)> f2;
    f2 = bind(&Test::data, _1);
    f2(t1) = 20;                    // ok
    f2(t2) = 40;                    // ok
}
```

---

## Example

```cpp
#include <iostream>
#include <string>
#include <functional>
#include <map>
#include <vector>
using namespace std;
using namespace std::placeholders;

class NotificationCenter
{
    using HANDLER = function<void(void*)>;
    map<string, vector<HANDLER>> notif_map;
public:
    void Register(string key, HANDLER h)
    {
        notif_map[key].push_back(h);
    }
    void Notify(string key, void* param)
    {
        vector<HANDLER>& v = notif_map[key];

        for(auto f : v)
            f(param);
    }
};

void f1(void* p) { cout << "f1" << endl;}
void f2(void* p, int a, int b) { cout << "f2" << endl;}

int main()
{
    NotificationCenter nc;
    nc.Register("CONNECT_WIFI", &f1);
    nc.Register("CONNECT_WIFI", bind(&f2, _1, 0, 0));

    nc.Notify("CONNECT_WIFI", (void*)0);
}
```