---
layout: post
title:  "(C++ : Concurrency-1-3) detach thread"
summary: ""
author: C++
date: '2021-06-01 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/concurrency/1-3/
---

* [GetCode](https://github.com/EasyCoding-7/cpp_concurrency_masterclass)

---

## join and detach functions

* `detach` 
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