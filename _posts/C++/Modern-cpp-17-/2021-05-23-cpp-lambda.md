---
layout: post
title:  "(Modern C++ : 17~) Lambda"
summary: ""
author: C++
date: '2021-05-23 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/modern-cpp-17-/lambda/
---

결론부터 정리

## C++20의 lambda 추가기능

* 람다표현식에서 템플릿 사용 가능
* 평가 되지 않은 표현식에서 람다 표현식 사용가능
* 캠쳐하지 않은 람다 표현식에서 디폴트 생성자와 대입연사자 사용가능
* 암시적인 this, 캡쳐가 deprecated 됨
* parameter pack 캡쳐 가능

말로 설명하면 무슨말인지 모르니 아래에 설명참고

---

## 람다표현식에서 템플릿 사용 가능

```cpp
#include <iostream>

auto add1 = [](int a, int b) { return a + b; };
auto add2 = [](auto a, auto b) { return a + b; };
// generic lambda expression -> C++14부터 가능

int main()
{
  std::cout << add1(1, 2) << std::endl;       // 3
  std::cout << add1(1.1, 2.2) << std::endl;   // 3 -> error!

  std::cout << add2(1, 2) << std::endl;       // 3
  std::cout << add2(1.1, 2.2) << std::endl;   // 3.3
  std::cout << add2(1, 2.2) << std::endl;     // 3.2 -> 이걸 에러내고 싶다면??
  
  // 하고싶은것? -> 입력을 같은 타입만 받게 사용하고 싶다
}
```

```cpp
auto add3 = [](auto a, decltype(a) b) { return a + b; };

int main()
{
    std::cout << add3(1, 2) << std::endl;       // 3
    std::cout << add3(1.1, 2.2) << std::endl;   // 3.3
    std::cout << add3(1, 2.2) << std::endl;     // error!
}
```

이것을 **람다표현식에서 템플릿**로 처리

```cpp
#include <iostream>

auto add1 = [](auto a, auto b) { return a + b; };       // C++14
auto add2 = []<typename T>(T a, T b) { return a + b; }; // C++20

int main()
{
    std::cout << add1(1, 2) << std::endl;       // 3
    std::cout << add1(1.1, 2.2) << std::endl;   // 3.3
    std::cout << add1(1, 2.2) << std::endl;     // 3.2

    std::cout << add2(1, 2) << std::endl;       // 3
    std::cout << add2(1.1, 2.2) << std::endl;   // 3.3
    std::cout << add2(1, 2.2) << std::endl;     // error!
}
```

---

## 평가 되지 않은 표현식에서 람다 표현식 사용가능

```cpp
#include <iostream>
#include <memory>

struct Freer
{
  inline void operator()(void* p) const noexcept
  {
    std::cout << "free" << std::endl;
    free(p);
  }
};

int main()
{
  std::unique_ptr<int> up1(new int);    // 안전하게 delete 가 불린다.

  //std::unique_ptr<int> up2(static_cast<int*>(malloc(100)));  // free를 통해야 delete가 된다.
  std::unique_ptr<int, Freer> up2(static_cast<int*>(malloc(100)));  // 소멸자를 지정해 줘야한다.
}
```

이걸 좀 람다표현식으로 간단하게 쓸 수 없나?

```cpp
std::unique_ptr<int, [](int* p){free(p);}> up2(static_cast<int*>(malloc(100)));
// 이렇게 표현하고 싶은데 그냥 람다식은 안들어가고 자료형이 들어가야한다.

std::unique_ptr<int, decltype([](int* p){free(p);})> up2(static_cast<int*>(malloc(100)));
// 이게 C++20부터는 선언가능
```

```cpp
#include <iostream>
#include <memory>

int add(int a, int b) { return a + b; }

int main()
{
  std::cout << sizeof(int) << std::endl;        // 4
  std::cout << sizeof(add(1,2)) << std::endl;   // 4 (리턴타임을 의미)

  decltype(add(1,2));   // int n : 표현식 결과의 타입

  std::cout << sizeof( [](int a, int b) { return a+b; }) << std::endl;  // 이건 뭐가 나올까?
  // C++17까지는 error
  // C++20은 동작한다.

  std::cout << sizeof( [](int a, int b) { return a+b; }) << std::endl;    // 1
  std::cout << sizeof( [](int a, int b) { return a+b; }(1,2)) << std::endl; // 4

}
```

---

## 캠쳐하지 않은 람다 표현식에서 디폴트 생성자와 대입연사자 사용가능

```cpp
#include <iostream>

int main()
{
  int v1 = 10;
  auto f1 = [](int a, int b){ return a + b; }

  //                        C++11 ~ C++17           C++20
  decltype(f1) f2;        // Error                  ok (디폴트 생성자)
  decltype(f1) f3 = f1;   // ok                     ok (복사 생성자)
  f3 = f1;                // Error                  ok (대입 연산자)
}
```

```cpp
#include <iostream>

int main()
{
  int v1 = 10;

  // 값을 받아 버린다면?
  auto f1 = [v1](int a, int b){ return a + b; }

  //                        C++11 ~ C++17           C++20
  decltype(f1) f2;        // Error                  Error (디폴트 생성자)
  decltype(f1) f3 = f1;   // ok                     ok (복사 생성자)
  f3 = f1;                // Error                  Error (대입 연산자)
}
```

---

## 암시적인 this, 캡쳐가 deprecated 됨

```cpp
#include <iostream>
#include <functional>

struct Sample
{
  int value = 0;
  auto foo()
  {
    int n = 10;

    // 멤버함수에서 [=]의 의미는 [=] 뿐만아니라 this까지 capture
    auto f = [=](int a) { return a + n + value; };  
    // value라는 멤버데이터는 this가 capture되기에 접근이 가능
    std::cout << sizeof(f) << std::endl;
    return f;
  }
};

std::function<int(int)> f;  // 전역으로 function pointer 선언함.

void goo()
{
  Sample s;
  f = s.foo();
}

int main() 
{ 
  goo(); 
  std::cout << f(10) << std::endl;    
  // C++17 - 컴파일은 되지만 쓰레기 값이 나온다. - goo에서 생성된 Sample은 소멸된지 오래!
  // C++20 - warning이 나타난다 - 암묵적 this를 쓰지말라고 경고나옴
}
```

---

## parameter pack 캡쳐 가능

```cpp
#include <iostream>

// Capture Parameter pack by value.
template<typename ... Args> auto f1(Args&&... args)
{
    return [...args = std:forward<Args>(args)](){ (std::cout << ... << args); };
}

// Capture Parameter pack by reference
template<typename ... Args> auto f2(Args&&... args)
{
    return [&...args = std::forward<Args>(args)](){ (std::cout << ... << args); };
}

int main()
{
    f1(1,2,3,4,5)();

    int a = 1, b = 2, c = 3;
    f2(a,b,c)();
}
```