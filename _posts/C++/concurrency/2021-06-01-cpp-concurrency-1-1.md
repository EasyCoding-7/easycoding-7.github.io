---
layout: post
title:  "(C++ : Concurrency-1-1) std::thread 사용하는 3가지 방법"
summary: ""
author: C++
date: '2021-06-01 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/concurrency/1-1/
---

* [GetCode](https://github.com/EasyCoding-7/cpp_concurrency_masterclass)

---


## How to launch a thread (std::thread사용하기)

```cpp
#include <iostream>
#include <thread>

void func1()
{
	std::cout << "Hello from function \n";
}

class my_class {

public:
	void operator()()
	{
		std::cout << "hello from the class with function call operator \n";
	}
};

int main()
{
    // 1. function으로 thread 생성
	std::thread thread1(func1);

    // 2. class operator로 thread 생성
	my_class mc;
	std::thread thread2(mc);

    // 3. lambda로 thread 생성
	std::thread thread3([] {
		std::cout << "hello from the lambda \n";
		});

	thread1.join();
	thread2.join();
	thread3.join();

	std::cout << "This is main thread \n";
}
```

```
hello from the class with function call operator
Hello from function
hello from the lambda
This is main thread
```

---

## 추가) sleep사용하기

```cpp
#include <iostream>
#include <thread>
#include <chrono>
using namespace std;

int main()
{
    thread::id id = this_thread::get_id();

    cout << id << endl;

    this_thread::sleep_for(3s);
    this_thread::sleep_until(chrono::system_clock::now()+3s);
    this_thread::yield();
}
```