---
layout: post
title:  "(C++) thread"
summary: ""
author: C++
date: '2021-03-26 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
keywords: ['thread']
usemathjax: true
permalink: /blog/cpp/thread/
---

## STL thread의 목적

기존에 OS별 별도로 돌던 thead를 하나로 합쳐보자!<br>
단, C++표준이기에 모든 OS에서 돌아가게 만들었다.<br>
그에 따른 쓰기 불편한 점이 몇 가지 존재한다..<br>

---

## 기존의 Thread 정리

얼마나 불편했는지 보자

```cpp
// Windwos
#include <Windows.h>

DWORD WINAPI PrintMessage()
{
    // ...
}

int main() {
    DWORD myThreadID;

    HANDLE myHandle = CreateThread(0, 0, PrintMessage, NULL, 0, &myThreadID);

    WaitForSingleObject(myHandle, INFINITE);
    // thread가 종료될때 까지 기다린다.

    CloseHandle(myHandle);

    return 0;
}
```

```cpp
// POSIX(pthread)
#include <pthread.h>

void *printMessage()
{
    // ...
}

int main() {
    pthread_t thread = 0;

    int result_code = pthread_create(&thread, NULL, printMessage, NULL);
    result_code = pthread_join(thread, NULL);

    return 0;
}
```

사용자는 이걸 다 알아야 한다.

C++11 전까지 표준 멀티쓰레딩 라이브러리가 없음<br>
OS마다 멀티쓰레딩 구현이 달랐음<br>

* 리눅스/유닉스 : POSIX(pthread)
* 윈도우 쓰레드
        - 윈도우에서 pthread를 사용할순 있음(ver 1003.1만)
        - 불편하네… 표준 Thread를 만들어 보자

---

## thread 생성

```cpp
#include <iostream>
#include <string>
#include <thread>

void PrintMessage(const std::string& message)
{
    std::cout << message << std::endl;
}

int main() {
    std::thread thread(PrintMessage, "Message from a child thread");
    
    PrintMessage("Waiting the child thread");

    // 현재 스레드에서 떼어 내기
    thread.detach();

    // 다시 붙이고 싶다면?
    if(thread.joinable())
    {
        thread.join();
    }

    return 0;
}
```

```cpp
// 람다식 사용

#include <iostream>
#include <string>
#include <thread>


int main() {
    auto PrintMessage = [](const std::string& message)
    {
        std::cout << message << std::endl;
    };

    std::thread thread(PrintMessage, "Message from a child thread");
    
    PrintMessage("Waiting the child thread");

    thread.join();

    return 0;
}
```

```cpp
// 람다식 스레드에 매개변수 넣기

#include <iostream>
#include <string>
#include <thread>
#include <vector>

int main() {
    std::vector<int> list(100, 1);
    int result = 0;

    std::thread thread([](const std::vector<int>& v, int& result)
    {
        for( auto item : v )
        {
            result += item;
        }
    }, list, std::ref(result));

    thread.join();

    std::cout << "Result: " << result << std::endl;

    return 0;
}
```

## thread에 매개변수 넣기

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

```cpp
// thread 잠깐 정지

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
    this_thread::yield();           // 다른 스레드에게 우선순위를 양보한다.
}
```

```cpp
#include <iostream>
#include <thread>
using namespace std;

void foo()
{
    cout << "thread start" << endl;
    this_thread::sleep_for(2s);
    cout << "thread end" << endl;
}

int main()
{
    thread t(&foo);
    t.join();       // 스레드 종료를 대기
    // or
    // t.detach();
}
```

```cpp
#include <iostream>
#include <thread>
using namespace std;

void f1() {}
void f2(int a) {}

struct Worker
{
    void Main() {}
};

struct Functor
{
    void operator()() {}
};

int main()
{
    thread t1(&f1);

    thread t2(&f2, 5);

    Worker w;
    thread t3(&Workder::Main, &w);

    Functor f;
    thread t4(f);

    thread t5([](){cout << "thread t5" << endl;});
}
```

