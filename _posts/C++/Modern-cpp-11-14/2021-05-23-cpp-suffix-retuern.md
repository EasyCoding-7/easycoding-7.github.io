---
layout: post
title:  "(Modern C++ : 11~14) 후위리턴(-> return type)"
summary: ""
author: C++
date: '2021-05-23 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/modern-cpp-11-14/suffix-retuern/
---

## Suffix Return 이란?

```cpp
// int suqare(int a)
auto square(int a) -> int   // 후위 반환 타입
{
  return a * a;
}

int main()
{
  square(3);
}
```

어디다 쓰나?

```cpp
/*
int add(int a, int b)
{
  return a + b;
}
*/
template<typename T>
T add(T a, T b)
{
  return a + b;
}

int main()
{
  add(1, 2);    // ok
  add(1, 1.2);  // compile error
}
```

```cpp
template<typename T1, typaname T2>
?? add(T1 a, T2 b) // 리턴형은 뭘로 선언해야하나??
{
  return a + b;
}
```

```cpp
template<typename T1, typaname T2>
auto add(T1 a, T2 b) -> decltype(a+b)   // C++11
auto add(T1 a, T2 b)  // C++14
{
  return a + b;
}
```

---

## 추가적으로 어디쓸 수 있는지?

```cpp
auto text() {
  return 7;   // 이런것도 가능
}

auto text() -> int {
  return 7;   // 직접 지정도 가능
}

// 보통은 아래와 같이 쓴다
template <class T>
auto test(T value) -> decltype(value) {
  return value;
}

template <class T, class S>
auto test(T value, S value2) -> decltype(value + value2) {
  return value + value2;
  // 함수 내부의 결과에 따라 리턴타입을 결정
}

int get() {
  return 999;
}

// 함수를 리턴할때 auto를 쓰고 싶다면 이렇게 쓰자
auto test2() -> decltype(get()) {
  return get();
}
```

```cpp
int x = 10;
int foo(int a) { return x; }

// 리턴 타입을 아래와 같이 지정하고 싶지만 ... f의 함수 선언시점 보다 리턴 시점에서 사용하는게 빨라서 error
template<typename F, typename T>
decltype(f(std::forward<T>(arg))) chronometry(F f, T&& arg)
{
   return f(std::forward<T>(arg));
}
```

```cpp
template<typename F, typename T>
auto chronometry(F f, T&& arg) -> decltype(f(std::forward<T>(arg))) 
{
   return f(std::forward<T>(arg));
}
```