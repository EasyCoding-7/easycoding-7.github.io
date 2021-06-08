---
layout: post
title:  "(C++ STL) thread-async (concurrency 기초2)"
summary: ""
author: C++
date: '2021-06-08 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/stl/thread-async/
---

```cpp
#include <iostream>
#include <thread>
#include <future>
using namespace std;

int f1()
{
    this_thread::sleep_for(3s);
    return 10;
}

int main()
{
    // thread t(&f1);
    // t.join();

    // async로 thread를 대신해보자.
    future<int> ft = async(launch::async, &f1);
    cout << "Main routine" << endl;

    cout << ft.get() << endl;   // 스레드 종료 대기
}
```

* launch policy
    * `async` : 비동기로 실행
    * `defered` : 지연된 실행
    * `async | deferred` : 스레드 생성이 가능하면 스레드로 그렇지 않으면 지연된 실행

```cpp
#include <iostream>
#include <thread>
#include <future>
using namespace std;

int f1()
{
    cout << "f1 : " << this_thread::get_id() << endl;
    this_thread::sleep_for(3s);
    return 10;
}

int main()
{
    cout << "Main : "  << this_thread::get_id() << endl;

    // future<int> ft = async(launch::async, &f1);
    future<int> ft = async(launch::defered, &f1);
    this_thread::sleep_for(1s);
    cout << "Main routine" << endl;

    cout << ft.get() << endl;       // defered 옵션이라면, get호출 시점에서 thread 생성
}
```

```cpp
#include <iostream>
#include <thread>
#include <future>
using namespace std;

int f1()
{
    this_thread::sleep_for(3s);
    return 10;
}

int main()
{
    // future<int> ft = async(launch::async, &f1);
    // 리턴을 받지 않는다면?
    async(launch::async, &f1);  // future가 임시 객체로 리턴
    // 이 줄이 종료되면 리턴(임시객체)는 삭제된다.
    // 따라서 스레드가 실행되며 3초간 이 줄에서 대기하게 된다.

    cout << "Main routine" << endl;

    //cout << ft.get() << endl;
}
```