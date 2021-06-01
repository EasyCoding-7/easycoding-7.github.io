---
layout: post
title:  "(C++ : Concurrency-1-7) std::thread의 유용한 함수"
summary: ""
author: C++
date: '2021-06-01 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/concurrency/1-7/
---

* [GetCode](https://github.com/EasyCoding-7/cpp_concurrency_masterclass)

---

## Some useful operations on thread

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void run_code1()
{
	std::cout << "Allowed max number of parallel threads : "
		<< std::thread::hardware_concurrency() << std::endl;
}

void func_1()
{
	std::this_thread::sleep_for(std::chrono::milliseconds(1000));
	std::cout << "new  thread id : " << std::this_thread::get_id() << std::endl;
}

void run_code2()
{
	std::thread thread_1(func_1);

	std::cout << "thread_1 id before joining : " << thread_1.get_id() << std::endl;
	thread_1.join();

	std::thread thread_2;

	std::cout << "default consturcted thread id : " << thread_2.get_id() << std::endl;
	std::cout << "thread_1 id after joining : " << thread_1.get_id() << std::endl;
	std::cout << "Main thread id : " << std::this_thread::get_id() << std::endl;

	std::cout << "\n\nAllowed max number of parallel threads : "
		<< std::thread::hardware_concurrency() << std::endl;
}

int main()
{
	run_code1();	 // example 1
	//run_code2();	 // example 2
}
```