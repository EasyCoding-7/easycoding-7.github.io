---
layout: post
title:  "(C++ STL) thread-sync (concurrency 기초1)"
summary: ""
author: C++
date: '2021-06-08 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/stl/thread-sync/
---

```cpp
#include <iostream>
#include <thread>
#include <string>
#include <mutex>
using namespace std;

mutex m;
int global = 0;

void f1()
{
    lock_guard<mutex> lg(m);
    // lock_guard는 mutex내에서 exception이 생길경우를 처리해준다.
    // unlock은 소멸자에서 자동으로 해준다.
    //m.lock();
    global = 100;
    global = global + 1;
    //m.unlock();
}

int main()
{
    thread t1(&f1);
    thread t2(&f1);

    t1.join();
    t2.join();
}
```

```cpp
#include <iostream>
#include <thread>
#include <future>
using namespace std;

void f1(promise<int>& p)
{
    this_thread::sleep_for(3s);
    p.set_value(10);
}

int main()
{
    promise<int> p;
    future<int> ft = p.get_future();

    thread t(&f1, ref(p));
    cout << "wait value " << endl;
    cout << "value : " << ft.get() << endl; // set value까지 대기한다.

    t.join();
}
```