---
layout: post
title:  "(C++ : Concurrency-1-4) thread guard 만들어보기"
summary: ""
author: C++
date: '2021-06-01 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/concurrency/1-4/
---

* [GetCode](https://github.com/EasyCoding-7/cpp_concurrency_masterclass)

---

## How to handle join, in exception scenarios

여기서 하고싶은 말은 메인 thread가 종료되기 전 자신이 관리하는 thread를 대기하게 만들고 싶다

```cpp
#include <iostream>
#include <thread>
#include <stdexcept>
#include <chrono>
#include "common_objs.h"

void func_1()
{
	std::this_thread::sleep_for(std::chrono::milliseconds(500));
	std::cout << "hello from method \n";
}

void other_operations()
{
	std::cout << "This is other operation \n";
	throw std::runtime_error("this is a runtime error");
}

/////////////////////////////// for first example
void run_code1()
{
	std::thread thread_1(func_1);

	try {
		//do other operations
		other_operations();     // 여기서 error를 throw한다면?
		thread_1.join();        // 여기서 대기를 하고싶은데
	}
	catch (...)
	{
	}
	std::cout << "This is main thread \n";
}

/////////////////////////////// for second example
void run_code2()
{
	std::thread thread_1(func_1);

	//do other operations
	try
	{
		other_operations();
		thread_1.join();
	}
	catch (...)
	{
		thread_1.join();        // throw자체에서 대기할까? -> 매번이걸 한다고??
	}
}

/////////////////////////////// for third example
void run_code3()
{
	std::thread thread_1(func_1);
	thread_guard tg(thread_1);  // thread guard를 생성해보자.

	//do other operations
	try
	{
		other_operations();
	}
	catch (...)
	{
	}
}


int main()
{
	run_code1();
	//run_code2();
	//run_code3();
}
```

`thread_guard`의 내부는 이러하다

```cpp
class thread_guard
{
    std::thread& t;
    public:
    explicit thread_guard(std::thread& t_): t(t_){}
    ~thread_guard()
    {
        if(t.joinable())
        {
            // 내부에서 종료를 대기해준다.
            t.join();
        }
    }
    thread_guard(thread_guard const&)=delete;
    thread_guard& operator=(thread_guard const&)=delete;
};
```