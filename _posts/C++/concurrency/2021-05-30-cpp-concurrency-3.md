---
layout: post
title:  "(C++ : Concurrency-3) Communication between thread using condition variables and futures"
summary: ""
author: C++
date: '2021-05-30 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/concurrency/3/
---

* [GetCode](https://github.com/EasyCoding-7/cpp_concurrency_masterclass)

---

* [참고사이트](https://modoocode.com/284)

## introduction to condition variables

```cpp
#include <iostream>
#include <mutex>
#include <thread>
#include <string>
#include <thread>
#include <chrono>
#include <condition_variable>

bool have_i_arrived = false;
int total_distance = 5;
int distance_coverd = 0;
std::condition_variable cv;
std::mutex m;

void keep_moving()
{
	while (true)
	{
		std::this_thread::sleep_for(std::chrono::milliseconds(1000));
		distance_coverd++;

		//notify the waiting threads if the event occurss
		if (distance_coverd == total_distance)
			cv.notify_one();
	}

}

void ask_driver_to_wake_u_up_at_right_time()
{
	std::unique_lock<std::mutex> ul(m);
    // 함수의 조건이 만족할때 동작함.
	cv.wait(ul, [] {return distance_coverd == total_distance; });
	std::cout << "finally i am there, distance_coverd = " << distance_coverd << std::endl;;
}

int main()
{
	std::thread driver_thread(keep_moving);
	std::thread passener_thread(ask_driver_to_wake_u_up_at_right_time);
	passener_thread.join();
	driver_thread.join();
}
```

---

## Thread safe queue implementation

```cpp
#pragma once
#include <iostream>
#include <mutex>
#include <queue>
#include <memory>
#include <condition_variable>
#include <thread>

template<typename T>
class thread_safe_queue {
	std::mutex m;
	std::condition_variable cv;
	std::queue<std::shared_ptr<T>> queue;

public:
	thread_safe_queue()
	{}

	thread_safe_queue(thread_safe_queue const& other_queue)
	{
		std::lock_guard<std::mutex> lg(other_queue.m);
		queue = other_queue.queue;
	}

	void push(T& value)
	{
		std::lock_guard<std::mutex> lg(m);
		queue.push(std::make_shared<T>(value));
		cv.notify_one();
	}

	std::shared_ptr<T> pop()
	{
		std::lock_guard<std::mutex> lg(m);
		if (queue.empty())
		{
			return std::shared_ptr<T>();
		}
		else
		{
			std::shared_ptr<T> ref(queue.front());
			queue.pop();
			return ref;
		}
	}

	bool empty()
	{
		std::lock_guard<std::mutex> lg(m);
		return queue.empty();
	}

	std::shared_ptr<T> wait_pop()
	{
		std::unique_lock<std::mutex> lg(m);
		cv.wait(lg, [this] {
			return !queue.empty();
			});
		std::shared_ptr<T> ref = queue.front();
		queue.pop();
		return ref;
	}

	size_t size()
	{
		std::lock_guard<std::mutex> lg(m);
		return queue.size();
	}

	bool wait_pop(T& ref)
	{
		std::unique_lock<std::mutex> lg(m);
		cv.wait(lg, [this] {
			return !queue.empty();
			});

		ref = *(queue.front().get());
		queue.pop();
		return true;
	}

	bool pop(T& ref)
	{
		std::lock_guard<std::mutex> lg(m);
		if (queue.empty())
		{
			return false;
		}
		else
		{
			ref = queue.front();
			queue.pop();
			return true;
		}
	}
};
```

---

## indroduction to futures and async tasks

```cpp
#include <iostream>
#include <future>

int find_answer_how_old_universe_is()
{
	//this is not the ture value
	return 5000;
}

void do_other_calculations()
{
	std::cout << "Doing other stuff " << std::endl;
}

int main()
{
	std::future<int> the_answer_future = std::async(find_answer_how_old_universe_is);
	do_other_calculations();
	std::cout << "The answer is " << the_answer_future.get() << std::endl;
}
```

```cpp
#include <iostream>
#include <future>
#include <string>

void printing()
{
	std::cout << "printing runs on-" << std::this_thread::get_id() << std::endl;
}

int addition(int x, int y)
{
	std::cout << "addition runs on-" << std::this_thread::get_id() << std::endl;
	return x + y;
}

int substract(int x, int y)
{
	std::cout << "substract runs on-" << std::this_thread::get_id() << std::endl;
	return x - y;
}

int main()
{
	std::cout << "main thread id -" << std::this_thread::get_id() << std::endl;

	int x = 100;
	int y = 50;

    // std::launch::async - 바로 스레드 생성 후 실행
	std::future<void> f1 = std::async(std::launch::async, printing);
    // std::launch::deferred - get이 호출되면 스레드 생성 후 실행
	std::future<int> f2 = std::async(std::launch::deferred, addition, x, y);
	std::future<int> f3 = std::async(std::launch::deferred | std::launch::async,
		substract, x, y);

	f1.get();
	std::cout << "value recieved using f2 future -" << f2.get() << std::endl;
	std::cout << "value recieved using f2 future -" << f3.get() << std::endl;

}
```

---

## Parallel accumulate algorithm implementation with async task

```cpp
#include <iostream>
#include <future>
#include <numeric>

int MIN_ELEMENT_COUNT = 1000;

template<typename iterator>
int parallal_accumulate(iterator begin, iterator end)
{
	long length = std::distance(begin, end);

	//atleast runs 1000 element
	if (length <= MIN_ELEMENT_COUNT)
	{
		std::cout << std::this_thread::get_id() << std::endl;
		return std::accumulate(begin, end, 0);
	}

	iterator mid = begin;
	std::advance(mid, (length + 1) / 2);

	//recursive all to parallel_accumulate
	std::future<int> f1 = std::async(std::launch::deferred | std::launch::async,
		parallal_accumulate<iterator>, mid, end);
	auto sum = parallal_accumulate(begin, mid);
	return sum + f1.get();
}

int main()
{
	std::vector<int> v(10000, 1);
	std::cout << "The sum is " << parallal_accumulate(v.begin(), v.end()) << '\n';
}
```

---

## introduction to package_task

```cpp
#include <iostream>
#include <future>
#include <numeric>
#include <thread>
#include <functional>

int add(int x, int y)
{
	std::this_thread::sleep_for(std::chrono::milliseconds(500));
	std::cout << "add function runs in : " << std::this_thread::get_id() << std::endl;
	return x + y;
}

void task_thread()
{
	std::packaged_task<int(int, int)> task_1(add);
	std::future<int> future_1 = task_1.get_future();

	std::thread thread_1(std::move(task_1), 5, 6);
	thread_1.detach();

	std::cout << "task thread - " << future_1.get() << "\n";
}

void task_normal()
{
	std::packaged_task<int(int, int)> task_1(add);
	std::future<int> future_1 = task_1.get_future();
	task_1(7, 8);
	std::cout << "task normal - " << future_1.get() << "\n";
}

int main()
{
	task_thread();
	task_normal();
	std::cout << "main thread id : " << std::this_thread::get_id() << std::endl;
}
```

---

## Communication between thread using std::promies

```cpp
#include <iostream>       
#include <functional>     
#include <thread>        
#include <future>       
#include <stdexcept>

void print_int(std::future<int>& fut) {
	std::cout << "waiting for value from print thread \n";
	std::cout << "value: " << fut.get() << '\n';
}

int main()
{
	std::promise<int> prom;
	std::future<int> fut = prom.get_future();

	std::thread print_thread(print_int, std::ref(fut));

	std::this_thread::sleep_for(std::chrono::milliseconds(5000));
	std::cout << "setting the value in main thread \n";
	prom.set_value(10);

	print_thread.join();
}
```

---

## Retrieving exception using std::futures

```cpp
#include <iostream>       
#include <thread>         
#include <future>         
#include <stdexcept>   

void throw_exception()
{
	throw  std::invalid_argument("input cannot be negative");
}

void calculate_squre_root(std::promise<int>& prom)
{
	int x = 1;
	std::cout << "Please, enter an integer value: ";
	try
	{
		std::cin >> x;
		if (x < 0)
		{
			throw_exception();
		}
		prom.set_value(std::sqrt(x));
	}
	catch (std::exception&)
	{
		prom.set_exception(std::current_exception());
	}
}

void print_result(std::future<int>& fut) {
	try
	{
		int x = fut.get();
		std::cout << "value: " << x << '\n';
	}
	catch (std::exception& e) {
		std::cout << "[exception caught: " << e.what() << "]\n";
	}
}

int main()
{
	std::promise<int> prom;
	std::future<int> fut = prom.get_future();

	std::thread printing_thread(print_result, std::ref(fut));
	std::thread calculation_thread(calculate_squre_root, std::ref(prom));

	printing_thread.join();
	calculation_thread.join();
}
```

---

## std::shared_futures

```cpp
#include <iostream>       
#include <thread>         
#include <future>         
#include <stdexcept>   
#include <chrono>
#include <mutex>

/******************************* Example 1 ******************************/
void print_result1(std::future<int>& fut)
{
	//std::cout << fut.get() << "\n";
	if (fut.valid())
	{
		std::cout << "this is valid future\n";
		std::cout << fut.get() << "\n";
	}
	else
	{
		std::cout << "this is invalid future\n";
	}
}

void run_code1()
{
	std::promise<int> prom;
	std::future<int> fut(prom.get_future());

	std::thread th1(print_result1, std::ref(fut));
	std::thread th2(print_result1, std::ref(fut));

	prom.set_value(5);

	th1.join();
	th2.join();
}

/************************************* Example 2 **************************************/

void print_result2(std::shared_future<int>& fut)
{
	std::cout << fut.get() << "  - valid future \n";
}

void run_code2()
{
	std::promise<int> prom;
	std::shared_future<int> fut(prom.get_future());

	std::thread th1(print_result2, std::ref(fut));
	std::thread th2(print_result2, std::ref(fut));

	prom.set_value(5);

	th1.join();
	th2.join();
}

int main()
{
	run_code1();
	run_code2();

	return 0;
}

```