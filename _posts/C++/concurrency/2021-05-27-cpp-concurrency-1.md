---
layout: post
title:  "(C++ : Concurrency-1) Thread management guide"
summary: ""
author: C++
date: '2021-05-27 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/concurrency/1/
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

```
this is joinable thread
this is joinable thread
hello from main thread
hello from method
```

---

## join and detach functions

* `detash` 
    * 오브젝트와 스레드를 해제한다. 
    * detach 함수는 thread 오브젝트에 연결된 스레드를 떼어냅니다.
    * 그래서 detach 이후에는 thread 오브젝트로 스레드를 제어할 수 없습니다
    * 그러나 **detach를 호출했다고 해서 관련된 스레드가 종료 되는 것이 아닙니다**
    * 즉 thread 오브젝트와 스레드의 **연결이 끊어지는 것**입니다.
        * [참고사이트](http://blog.naver.com/PostView.nhn?blogId=icarus_buk&logNo=220951335714)

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void func_1()
{
	std::this_thread::sleep_for(std::chrono::milliseconds(5000));
	std::cout << "hello from func_1 \n";
}


int main()
{
	std::thread thread_1(func_1);
	thread_1.join();        
	// thread_1.detach();   // detach시 func_1()은 불리지 않는다.

	std::cout << "hello from main thread \n";
}
```

```cpp
    // ...

    first.detach();
 
    // 스레드 아이디가 초기화 된다.
    std::cout << "after detach first thread id : " << first.get_id() << endl;
            
    // 스레드가 멈출때 까지 대기
    if (true == first.joinable()) // detach후 joinable하지 않으면 에러 발생
        first.join();                
    if (true == second.joinable())
        second.join();  

    // ...
```

---

## How to handle join, in exception scenarios

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
		other_operations();
		thread_1.join();
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
		thread_1.join();
	}
}

/////////////////////////////// for third example
void run_code3()
{
	std::thread thread_1(func_1);
	thread_guard tg(thread_1);

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
        t.join();
    }
    }
    thread_guard(thread_guard const&)=delete;
    thread_guard& operator=(thread_guard const&)=delete;
};
```

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

## Problematic situations may arise when passing parameters to a thread

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

## Transferring ownership of a thread

```cpp
#include <iostream>
#include <thread>

void func_1()
{
	std::cout << "This is a function 1";
}

void func_2()
{
	std::cout << "This is a function 2";
}

int main()
{
	std::thread thread_1(func_1);

	//try to assigne one thread to another
	//std::thread thread_2 = thread_1;  

	//move one thread form another
	std::thread thread_2 = std::move(thread_1);

	//implicit call to move constructor
	thread_1 = std::thread(func_2);

	std::thread thread_3 = std::move(thread_2);

	//this is wrong : 생성시에만 가능.
	thread_1 = std::move(thread_3);

	thread_1.join();
	thread_3.join();

}
```

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

---

## Parallel accumulate - algorithm explanation

```cpp
// std::accumulate 역할
void sequntial_accumulate_test()
{
	std::vector<int> v{1,2,3,4,5,6,7,8,9,10};
	int sum = std::accumulate(v.begin(), v.end(), 0);		// 덧셈 가능(함수를 별도로 기입안할경우 더하기)
	int product = std::accumulate(v.begin(), v.end(), 1, 
									std::multiplies<int>());	// 함수는 이걸써달라(곱셈)

	auto dash_fold = [](std::string a, int b)
	{
		return std::move(a) + "-" + std::to_string(b);
	};

	std::string s = std::accumulate(std::next(v.begin()), v.end(), // 여기서 여기까지
									std::to_string(v[0]), // 이 시작숫자로
									dash_fold);			  // 이 함수를 통해 accumulate

	std::cout << "sum - " << sum << "\n";
	std::cout << "product - " << product << "\n";
	std::cout << "dash fold - " << s << "\n";
}
```

```
sum - 55
product - 3628800
dash fold - 1-2-3-4-5-6-7-8-9-10
```

accumulate를 parrallel로 구현<br>
사용가능한 최대 Thread를 찾고 더하기 진행하는 코드라 생각

```cpp
#include <iostream>
#include <thread>
#include <stdexcept>

#include "parallel_accumulate.h"

int  main()
{
	int result = 0;
	std::vector<int> vec(10000);
	for (int i = 0; i < 10000; i++)
		vec[i] = 2;

	parallel_accumulate(vec.begin(), vec.end(), result);

	std::cout << "final result = " << result << std::endl;
}
```

```cpp
#pragma once

#include <thread>
#include <numeric>
#include <algorithm>
#include <vector>
#include <functional>

template<typename iterator, typename T>
void accumulate(iterator first, iterator last, T& val)
{
	val = std::accumulate(first, last, val);
}

template<typename iterator, typename T>
void parallel_accumulate(iterator start, iterator end, T& ref)
{
	unsigned MIN_BLOCK_SIZE = 1000;

	unsigned distance = std::distance(start, end);
	unsigned allowed_threads_by_elements = (distance + 1) / MIN_BLOCK_SIZE;
	unsigned allowed_threads_by_hardware = std::thread::hardware_concurrency();

	// this is due to the fact that some operating system may 
	// return 0 to indicate that this has not implemented
	if (allowed_threads_by_hardware < 1)
		allowed_threads_by_hardware = 2;

	unsigned allowed_threads = std::min(allowed_threads_by_elements,
		allowed_threads_by_hardware);

	//caculating the block size 
	unsigned block_size = (distance + 1) / allowed_threads;

	std::vector<T> results(allowed_threads);
	std::vector<std::thread> threads(allowed_threads - 1);

	//iterate and craeting new threads to calculate sum for each blocks
	iterator last;
	for (unsigned i = 0; i < allowed_threads - 1; i++)
	{
		last = start;
		std::advance(last, block_size);
		threads[i] = std::thread(accumulate<iterator, T>, start, last,
			std::ref(results[i]));
		start = last;
	}

	//final block will be calculated from this thread
	results[allowed_threads - 1] =
		std::accumulate(start, end, results[allowed_threads - 1]);

	//calling join on the newly craeted threads
	for_each(threads.begin(), threads.end(), std::mem_fn(&std::thread::join));
	ref = std::accumulate(results.begin(), results.end(), ref);
}
```

---

## debugging

```cpp
#include <iostream>
#include <thread>
#include <future>
#include <chrono>
#include <execution>
#include <string>

const size_t testSize = 1000;

// 이런게 있다 정도만 알자
using std::chrono::duration;
using std::chrono::duration_cast;
using std::chrono::high_resolution_clock;
using std::milli;

void function3()
{
	throw;
}


std::string function1(int i)
{
	function3();

	if (i < 100)
	{
		return std::string("Hello");
	}

	return std::string("Hi");
}



double function2( int i)
{
	std::string str = function1(i);
	
	if (i < 200)
	{
		return 3.98;
	}

	return 9.45;
}

int init_number(int i)
{
	function2(i);

	if (i < 1000)
	{
		i = i;
	}
	else
	{
		i = i * 2;
	}

	return i;
}

int main()
{

	std::vector<int> ints(testSize);
	for (size_t i = 0; i < testSize; i++)
	{
		ints[i] = init_number(i);
		std::cout << init_number(i) << " " << function1(i) << " " << function2(i);
	}

	return 0;
}
```