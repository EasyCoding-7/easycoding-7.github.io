---
layout: post
title:  "(C++ : Design-pattern) Proxy Pattern"
summary: ""
author: C++
date: '2021-04-02 0:00:00 +0000'
category: ['Cpp', 'Design-pattern']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/design-pattern/proxy-pattern/
---

## Proxy

어떤 객체에 대한 접근을 제어하기 위한 용도로 대리인이나 대변인에 해당하는 객체를 제공

예제를 통해 살펴보자

```cpp
// Server
#include <iostream>
#include "ecourse_dp.hpp"
using namespace std;
using namespace ecourse;

class Calc
{
public:
    int Add(int a, int b) { return a + b; }
    int Sub(int a, int b) { return a - b; }
};
Calc calc;

int dispatch(int code, int x, int y)
{
    printf("[DEBUG] %d, %d, %d\n", code, x, y);

    switch(code)
    {
        case 1: return calc.Add(x, y);
        case 2: return calc.Sub(x, y);
    }
    return -1;
}

int main()
{
    ec_start_server("CalcService", dispatch);
}
```

```cpp
// Client
#include <iostream>
#include "ecourse_dp.hpp"
using namespace std;
using namespace ecourse;

int main()
{
    int server = ec_find_server("CalcService");

    int ret = ec_send_server(server, 1, 10, 20);
    // 문제1) 연산명령이 숫자이니 사용자 입장에서 헷갈린다.
}
```

```cpp
class Calc      // 요놈이 Proxy이다.
{
    int server;
public:
    Calc() { server = ec_find_server("CalcService"); }

    int Add(int a, int b) { return ec_send_server(server, 1, a, b); }
    int Sub(int a, int b) { return ec_send_server(server, 2, a, b); }
};

int main()
{
    Calc* pCalc = new Calc;

    cout << pCalc->Add(1, 2) << endl;
}
```

* Proxy 장점
    * 명령 코드 대신 함수 호출 사용가능
    * 잘못된 명령 코드를 사용하지 않게 된다.
    * Client는 IPC에 대해서 알 필요가 없다.

코드정리

```cpp
struct ICalc
{
    virtual int Add(int a, int b) = 0;
    virtual int Sub(int a, int b) = 0;
    virtual ~ICalc();
}

// client
class Calc : public ICalc       
{
    int server;
public:
    Calc() { server = ec_find_server("CalcService"); }

    virtual int Add(int a, int b) { return ec_send_server(server, 1, a, b); }
    virtual int Sub(int a, int b) { return ec_send_server(server, 2, a, b); }
};

int main()
{
    Calc* pCalc = new Calc;

    cout << pCalc->Add(1, 2) << endl;
}

// server
class Calc : public ICalc       
{
public:
    int Add(int a, int b) { return a + b; }
    int Sub(int a, int b) { return a - b; }
};
```

이렇게 만들면 서버, 클라이언트 측 똑같은 이름의 함수를 만들게 강제할 수 있다. -> 이름을 통일해야 훗날 디버깅이 수월

```cpp
struct IRefCount
{
    virtual void AddRef() = 0;
    virtual void Release() = 0;
    virtual ~IRefCount() {}
};

struct ICalc : public IRefCount
{
    virtual int Add(int a, int b) = 0;
    virtual int Sub(int a, int b) = 0;

    virtual ~ICalc();
}
```

```cpp
// CalcProxy.cpp
#include "ecourse_dp.hpp"
#include "ICalc.h"
using namespace std;

class Calc : public ICalc
{
    int server;
    int count = 0;
public:
    Calc() { server = ec_find_server("CalcService"); }

    void AddRef() {++count;}
    void Release() {if(--count == 0) delete this;}

    int Add(int a, int b) { return ec_send_server(server, 1, a, b); }
    int Sub(int a, int b) { return ec_send_server(server, 2, a, b); }
};

extern "C" __declspec(dllexport)        // dll로 생성하고 함수 사용준비
ICalc* CreateCalc()
{
    return new Calc;
}
```

```cpp
typedef ICalc* (*F)();

int main()
{
    // 동적 모듈 load
    void* addr = ec_load_module("CalcProxy.dll");
    F f = (F)ec_get_function_address(addr, "CreateCalc");

    ICalc* pCalc = f();     // CreateCalc()
    pCalc->AddRef();

    cout << pCalcs->Add(1, 2);

    pCalc->Release();
}
```

물론 스마트 포인터로 만들면 굳이 AddRef, Release가 필요없긴하다.