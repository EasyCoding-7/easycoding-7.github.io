---
layout: post
title:  "(C++) decltype"
summary: ""
author: C++
date: '2021-03-26 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
keywords: ['decltype']
usemathjax: true
permalink: /blog/cpp/decltype/
---

## 좋은 Example

```cpp
int x = 10;

int& foo(int a, int b)
{
  return x;
}

int main()
{
  auto ret1 = foo(1, 2);      // int, auto는 참조형 타입을 제거하고 받음.
  decltype(foo(1,2))  ret2 = foo(1,2);   // int&, 평가되지 않은 표현식(unevaluated expression)
  
  //C++14
  decltype(auto) ret3 = foo(1,2);   // 위와 같은 표현이다.
}
```

리턴형이 애매할때 decltype으로 리턴변수를 받아쓰자. -> 그런데 이럴일이 있나?

```cpp
// 텔레그램의 소수중 하나
auto operator()(producer<Value, Error, Generator> &&initial) {  // operator()가 호출되면
    return make_producer<Value, Error>([                        // make_producer가 호출되는데, 그 매개변수로 람다식이 들어감
        initial = std::move(initial),
        limit = _count
    ](const auto &consumer) mutable {       
        auto count = consumer.template make_state<int>(limit);
        auto initial_consumer = make_consumer<Value, Error>(    // 세 개의 매개변수가 들어감.
        [consumer, count](auto &&value) {                       
            auto left = (*count)--;
            if (left) {
                consumer.put_next_forward(
                    std::forward<decltype(value)>(value));
                --left;
            }
            if (!left) {
                consumer.put_done();
            }
        }, [consumer](auto &&error) {
            consumer.put_error_forward(
                std::forward<decltype(error)>(error));          // 매개변수의 자료형을 알수 없기에 decltype을 사용하게 된다.
        }, [consumer] {
            consumer.put_done();
        });
        consumer.add_lifetime(initial_consumer.terminator());
        return std::move(initial).start_existing(initial_consumer);
    });
```

---

decltype을 설명하기 위해서 auto도 같이 설명이 들어가야한다. 일단 사용은

```cpp
int main()
{
    double x[5] = {1, 2, 3, 4, 5};

    // double n1 = x[0];
    auto n1 = x[0];
    decltype(n1) n3;
}
```

변수의 타입을 컴파일러가 결정하는 문법<br>
컴파일 시간에 결정이 되므로 실행시간에 오버헤드는 없다.<br>

## auto

* 우변의 수식으로 좌변의 타입을 결정한다.
* 반드시 초기값 (우변식)이 필요하다.

## decltype

* ()안의 표현식을 가지고 타입을 결정
* 초기값이 없어도 된다.

사실 여기까지만 봐서는 어디서 쓰일지 모르겠음... 예제로 보자

---

```cpp
#include <iostream>
#include <typeinfo>   // for typeid

int foo(int a, double d)
{
  return 0;
}

int main()
{
  foo(1, 3.1);
  
  decltype(foo) d1;             // 함수 타입 - int(int, double)
  decltype(&foo) d2;            // 함수 포인터 타입 - int(*)(int, double)
  decltype(foo(1, 3.1)) d3;     // 함수 반환(return) 타입 - int
  
  std::cout << typeid(d1).name() << std::endl;
  std::cout << typeid(d2).name() << std::endl;
  std::cout << typeid(d3).name() << std::endl;
}
```

```cpp
#include <typeinfo>
using namespace std;

int main() {
  int value;
  double dvalue

  cout << typeid(value).name() << endl;   // i
  cout << typeid(dvalue).name() << endl;  // d

  return 0;
}
```

```cpp
string value;

// 위에서 설명 됐듯 decltype의 ()안에 자료형으로 사용됨.
decltype(value) name = "Bob";

cout << name << endl; // okay
```

---

## 좀 더 상세한 설명

```cpp
int main()
{
  int n = 0;
  int* p = &n;
  
  decltype(n) d1;     // int
  decltype(p) d2;     // int*, 명확하게 자료형이 나오면 그 자료형 대로 판단
  
  decltype(*p) d3;     // int&, decltype(수식) : 수식이 lvalue라면 참조, 아니면 값 타입
  decltype((n)) d4;    // int&, 변수에 연산자("()")가 붙어있어 lvalue라 판단
  
  decltype(n+n) d5;     // int, n+n = 10 -> error
  decltype(++n) d6;     // int&, ++n = 10 -> ok
  decltype(n++) d7;     // int, n++ = 10 -> error
  
  int x[3] = {1,2,3};
  
  decltype(x[0]) d8;    // int&, x[0] = 10 -> ok
  auto a1 = x[0];       // int
  // 같은 x[0]라도 decltype과 auto의 차이를 알자
}
```

```cpp
int x = 10;

int& foo(int a, int b)
{
  return x;
}

int main()
{
  auto ret1 = foo(1, 2);      // int, auto는 참조형 타입을 제거하고 받음.
  decltype(foo(1,2))  ret2 = foo(1,2);   // int&, 평가되지 않은 표현식(unevaluated expression)
  
  //C++14
  decltype(auto) ret3 = foo(1,2);   // 위와 같은 표현이다.
}
```

---

## auto 와 decltype

```cpp
#include <iostream>
#include <typeinfo>
#include <vector>
using namespace std;

int main()
{
  // 배열
  int x[3] = { 1, 2, 3 };   // x : int[3]
  
  auto a1 = x;        // int a1[3] = x; - error, 따라서 배열의 첫 번째 요소 주소를 넣어준다.
  auto& a2 = x;       // int (&a2)[3] = x; - ok
                      // a2 : int (&)[3]
  
  decltype(x) d;      // int [3]
  cout << typeid(a1).name() << endl;   // int*
  cout << typeid(a2).name() << endl;   // int(&)[3]
  cout << typeid(d).name() << endl;   // int [3]
  
  auto a3 = 1;    // int
  auto a4{1};     // int
  auto a5 = {1};  // initializer_list
  
  vector<int> v1(10, 0);
  auto a6 = v1[0];      // int
  
  vector<bool> v2(10, 0);
  auto a7 = v2[0];      // _Bit_reference?? -> temporary proxy를 알아야 됨. 일단은 받아들이자.
}
```