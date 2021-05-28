---
layout: post
title:  "(C++ : Concurrency-2) Thread safe access to shared data and locking mechanisms"
summary: ""
author: C++
date: '2021-05-28 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/concurrency/2/
---

* [GetCode](https://github.com/EasyCoding-7/cpp_concurrency_masterclass)

---

## mutex

```cpp
#include <iostream>
#include <mutex>
#include <list>
#include <thread>

std::list<int> my_list;
std::mutex m;

// mutex사용방법이 2가지 있다.

void add_to_list1(int const& x)
{
	m.lock();
	my_list.push_front(x);
	m.unlock();
}

void size1()
{
	m.lock();
	int size = my_list.size();
	m.unlock();
	std::cout << "size of the list is : " << size << std::endl;
}


void add_to_list2(int const& x)
{
    // 이걸 추천
	std::lock_guard<std::mutex> lg(m);
	my_list.push_back(x);
}

void size2()
{
	std::lock_guard<std::mutex> lg(m);
	int size = my_list.size();
	std::cout << "size of the list is : " << size << std::endl;
}

int  main()
{
	std::thread thread_1(add_to_list2, 4);
	std::thread thread_2(add_to_list2, 11);

	thread_1.join();
	thread_2.join();
}
```

---

## Things to remember when using mutexes

```cpp
#include <iostream>
#include <mutex>
#include <list>
#include <thread>

/*********************** example 1 *********************/
class list_wrapper {
	std::list<int> my_list;
	std::mutex m;

public:
	void add_to_list(int const& x)
	{
		std::lock_guard<std::mutex> lg(m);
		my_list.push_front(x);
	}

	void size()
	{
		std::lock_guard<std::mutex> lg(m);
		int size = my_list.size();
		std::cout << "size of the list is : " << size << std::endl;
	}

	std::list<int>* get_data()
	{
		return &my_list;
	}
};

/*********************** example 2 *********************/
class data_object {

public:
	void some_operation()
	{
		std::cout << "this is some operation \n";
	}
};

class data_wrapper {

	data_object protected_data;
	std::mutex m;

public:
	template <typename function>
	void do_some_work(function f)
	{
		std::lock_guard<std::mutex> lg(m);
		f(protected_data);
	}
};

data_object* unprotected_data;

void malisious_function(data_object& data)
{
	unprotected_data = &data;
}

void run_code()
{
	data_wrapper wrapper;
	wrapper.do_some_work(malisious_function);
}


int main()
{
	run_code();
}
```

---

## Thread safe stack implementation

```cpp
#pragma once
#include <iostream>
#include <mutex>
#include <stack>
#include <thread>

template<typename T>
class trivial_thread_safe_stack {
	std::stack<T> stk;
	std::mutex m;

public:
	void push(T element)
	{
		std::lock_guard<std::mutex> lg(m);
		stk.push(element);
	}

	void pop()
	{
		std::lock_guard<std::mutex> lg(m);
		stk.pop();
	}

	T& top()
	{
		std::lock_guard<std::mutex> lg(m);
		return stk.top();
	}

	bool empty()
	{
		std::lock_guard<std::mutex> lg(m);
		return stk.empty();
	}

	size_t size()
	{
		std::lock_guard<std::mutex> lg(m);
		return stk.size();
	}
};
```

좀 더 완성도를 높여보자.

```cpp
#pragma once
#include <iostream>
#include <mutex>
#include <stack>
#include <thread>
#include <memory>
#include <stdexcept>

template<typename T>
class thread_safe_stack {
	std::stack<std::shared_ptr<T>> stk;
	std::mutex m;

public:
	void push(T element)
	{
		std::lock_guard<std::mutex> lg(m);
		stk.push(std::make_shared<T>(element));
	}

	std::shared_ptr<T> pop()
	{
		std::lock_guard<std::mutex> lg(m);
		if (stk.empty())
		{
			throw std::runtime_error("stack is empty");
		}

		std::shared_ptr<T> res(stk.top());
		stk.pop();
		return res;
	}

	void pop(T& value)
	{
		std::lock_guard<std::mutex> lg(m);
		if (stk.empty()) throw std::runtime_error("stack is empty");
		value = *(stk.top().get());
		stk.pop();
	}


	bool empty()
	{
		std::lock_guard<std::mutex> lg(m);
		return stk.empty();
	}

	size_t size()
	{
		std::lock_guard<std::mutex> lg(m);
		return stk.size();
	}
};
```

---

## deadlock

뭐 대략 mutex를 두 개쓰다보니 서로를 대기하다가 deadlock이 발생한다는 내용

```cpp
#include <iostream>
#include <mutex>
#include <thread>
#include <string>
#include <chrono>

/*********************************************** example 1 ***********************************/
class bank_account {
	double balance;
	std::string name;
	std::mutex m;


public:
	bank_account() {};

	bank_account(double _balance, std::string _name) :balance(_balance), name(_name) {}

	bank_account(bank_account& const) = delete;
	bank_account& operator=(bank_account& const) = delete;

	void withdraw(double amount)
	{
		std::lock_guard<std::mutex> lg(m);
		balance += amount;
	}

	void deposite(double amount)
	{
		std::lock_guard<std::mutex> lg(m);
		balance += amount;
	}

	void transfer(bank_account& from, bank_account& to, double amount)
	{
		std::lock_guard<std::mutex> lg_1(from.m);
		std::cout << "lock for " << from.name << " account acquire by " << std::this_thread::get_id() << std::endl;
		std::this_thread::sleep_for(std::chrono::milliseconds(1000));

		std::cout << "waiting to acquire lock for " << to.name << " account by  " << std::this_thread::get_id() << std::endl;
		std::lock_guard<std::mutex> lg_2(to.m);

		from.balance -= amount;
		to.balance += amount;
		std::cout << "transfer to - " << to.name << "   from - " << from.name << "  end \n";
	}

};

void run_code1()
{
	bank_account account;

	bank_account account_1(1000, "james");
	bank_account account_2(2000, "Mathew");

	std::thread thread_1(&bank_account::transfer, &account, std::ref(account_1), std::ref(account_2), 500);
	std::this_thread::sleep_for(std::chrono::milliseconds(100));
	std::thread thread_2(&bank_account::transfer, &account, std::ref(account_2), std::ref(account_1), 200);

	thread_1.join();
	thread_2.join();
}



/*********************************************** example 2 ***********************************/
std::mutex m1;
std::mutex m2;


void m1_frist_m2_second()
{
	std::lock_guard<std::mutex> lg1(m1);
	std::this_thread::sleep_for(std::chrono::milliseconds(1000));
	std::cout << "thread " << std::this_thread::get_id() << " has acquired lock for m1 mutex, its wait for m2 \n";
	std::lock_guard<std::mutex>lg2(m2);
	std::cout << "thread " << std::this_thread::get_id() << " has acquired lock for m2 mutex \n";
}


void m2_frist_m1_second()
{
	std::lock_guard<std::mutex> lg1(m2);
	std::this_thread::sleep_for(std::chrono::milliseconds(1500));
	std::cout << "thread " << std::this_thread::get_id() << " has acquired lock for m2 mutex, its wait for m1 \n";
	std::lock_guard<std::mutex>lg2(m1);
	std::cout << "thread " << std::this_thread::get_id() << " has acquired lock for m1 mutex \n";
}

void run_code2()
{
	std::thread thread_1(m1_frist_m2_second);
	std::thread thread_2(m2_frist_m1_second);

	thread_1.join();
	thread_2.join();
}


int main()
{
	run_code1();
	run_code2();

	return 0;
}
```

---

## unique_lock : 여러개의 mutex를 동시에 기다려달라 할 수 있다.

```cpp
#include <iostream>
#include <mutex>
#include <thread>
#include <string>


/***************************************************Example 1 ******************************************/
class bank_account {
	double balance;
	std::string name;
	std::mutex m;

public:
	bank_account() {};

	bank_account(double _balance, std::string _name) :balance(_balance), name(_name) {}

	bank_account(bank_account& const) = delete;
	bank_account& operator=(bank_account& const) = delete;

	void withdraw(double amount)
	{
		std::lock_guard<std::mutex> lg(m);
		balance += amount;
	}

	void deposite(double amount)
	{
		std::lock_guard<std::mutex> lg(m);
		balance += amount;
	}

	void transfer(bank_account& from, bank_account& to, double amount)
	{

		std::cout << std::this_thread::get_id() << " hold the lock for both mutex \n";

		std::unique_lock<std::mutex> ul_1(from.m, std::defer_lock);
		std::unique_lock<std::mutex> ul_2(to.m, std::defer_lock);
		std::lock(ul_1, ul_2);

		from.balance -= amount;
		to.balance += amount;
		std::cout << "transfer to - " << to.name << "   from - " << from.name << "  end \n";
	}
};

void run_code1()
{

	bank_account account;

	bank_account account_1(1000, "james");
	bank_account account_2(2000, "Mathew");

	std::thread thread_1(&bank_account::transfer, &account, std::ref(account_1), std::ref(account_2), 500);
	std::thread thread_2(&bank_account::transfer, &account, std::ref(account_2), std::ref(account_1), 200);

	thread_1.join();
	thread_2.join();
}



/***************************************************Example 2 ******************************************/

void x_operations()
{
	std::cout << "this is some operations \n";
}

void y_operations()
{
	std::cout << "this is some other operations \n";
}

std::unique_lock<std::mutex> get_lock()
{
	std::mutex m;
	std::unique_lock<std::mutex> lk(m);
	x_operations();
	return lk;
}

void run_code2()
{
	std::unique_lock<std::mutex> lk(get_lock());
	y_operations();
}

int main()
{
    // Example 1
	run_code1();

    // Example 2
	run_code2();

	return 0;
}
```