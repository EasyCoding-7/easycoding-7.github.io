---
layout: post
title:  "(C++ : Concurrency-1-8) Thread accumulate Example"
summary: ""
author: C++
date: '2021-06-01 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/concurrency/1-8/
---

* [GetCode](https://github.com/EasyCoding-7/cpp_concurrency_masterclass)

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
