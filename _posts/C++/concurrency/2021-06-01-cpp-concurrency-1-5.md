---
layout: post
title:  "(C++ : Concurrency-1-5) Thread에 매개변수 넣기"
summary: ""
author: C++
date: '2021-06-01 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/concurrency/1-5/
---

* [GetCode](https://github.com/EasyCoding-7/cpp_concurrency_masterclass)

---

## How to pass parameters to a thread

```cpp
#include <iostream>
#include <thread>
#include <chrono>

////////////////////////////// for first example
void func_1(int x, int y)
{
	std::cout << " X + Y = " << x + y << std::endl;
}

void run_code1()
{
	int p = 9;
	int q = 8;


    // 함수는 매개변수를 이렇게 전달
	std::thread thread_1(func_1, p, q);

	thread_1.join();
}


////////////////////////////// for second example
void func_2(int& x)
{
	while (true)
	{
		std::cout << "Thread_1 x value : " << x << std::endl;
		std::this_thread::sleep_for(std::chrono::milliseconds(1000));
	}
}

void run_code2()
{
	int x = 9;
	std::cout << "Main thread current value of X is : " << x << std::endl;

    // 참조형으로도 전달가능
	std::thread thread_1(func_2, std::ref(x));
	std::this_thread::sleep_for(std::chrono::milliseconds(5000));

	x = 15;
	std::cout << "Main thread X value changed to : " << x << std::endl;
	thread_1.join();
}


int main()
{
	run_code1();
	//run_code2();
}
```

---

## 주의사항

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void func_2(int& x)
{
	while (true)
	{
		std::cout << x << std::endl;
		std::this_thread::sleep_for(std::chrono::milliseconds(1000));
	}
}

void func_1()
{
	int x = 5;

    // 참조형으로 넘겼는데
	std::thread thread_2(func_2, std::ref(x));
	thread_2.detach();
	std::this_thread::sleep_for(std::chrono::milliseconds(5000));

    // 이 thread가 종료되어 버린다면?? -> func_2에서 참조하지 못함
	std::cout << "thread_1 finished execution \n";
}


int main()
{
	std::thread thread_1(func_1);
	thread_1.join();
}
```

---

## 추가

```cpp
#include <iostream>
#include <thread>
using namespace std;

void f1(int a, int b)
{

}

int main()
{
    thread t1(&f1, 1, 2);
    thread t2(bind(&f1, 1, 2));
    // 둘 다 동일한 표현이다.

    t1.join();
    t2.join();
}
```

```cpp
int n = 10;
thread t1(&f1, 1, n);            // error
thread t1(&f1, 1, ref(n));       // ok
```