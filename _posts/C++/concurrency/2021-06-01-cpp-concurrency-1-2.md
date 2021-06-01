---
layout: post
title:  "(C++ : Concurrency-1-2) joinabiligy of threads"
summary: ""
author: C++
date: '2021-06-01 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/concurrency/1-2/
---

* [GetCode](https://github.com/EasyCoding-7/cpp_concurrency_masterclass)

---

## joinabiligy of threads

```cpp
#include <iostream>
#include <thread>

void func_1()
{
	std::cout << "hello from method \n";
}

int main()
{
	std::thread thread_1(func_1);   
    // 이 thread의 시작 시점이 궁금할 수 있다
    // 생성과 동시에 바로 시작됨을 기억하자(join이 호출돼야 실행되는게 아님)

	if (thread_1.joinable())
		std::cout << "this is joinable thread \n";

    // 여기서 join시 joinable == false
	//thread_1.join();

	if (thread_1.joinable())
	{
		std::cout << "this is joinable thread \n";
	}
	else
	{
		std::cout << "after calling join, thread_1 is not a joinable thread \n";
	}

	std::cout << "hello from main thread \n";
}
```
