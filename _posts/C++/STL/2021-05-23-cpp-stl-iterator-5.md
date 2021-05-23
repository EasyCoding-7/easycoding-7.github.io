---
layout: post
title:  "(C++ STL : iterator-5) iterator_traits"
summary: ""
author: C++
date: '2021-05-23 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/stl/iterator-5/
---

```cpp
#include <iostream>
#include <list>
using namespace std;

template<typename T>
typename T::value_type sum(T first, T last)
// 말 그대로 value_type을 말한다. (vector면 vector, list이면 list ...)
{
    typename T::value_type s = 0;
    // auto s = *first;     // C++11이후 버전은 이렇게 사용

    while(first != last)
    {
        s = s + *first;
        ++first;
    }
}

int main()
{
    list<int> s = {1,2,3,4,5,6,7,8,9,10};

    int n = sum(begin(s), end(s));

    cout << n << endl;
}
```

---

위 코드에는 심각한 문제가 있다.<br>
만약, list가 가지 않고 진짜 배열이 들어간다면??<br>
T::value_type을 찾을 수 없다.<br>

```cpp
template<typename T> struct iterator_traits
{
    using value_type = typename T::value_type;
};

template<typename T> struct iterator_traits<T*>     // 부분특수화가 가능!
{
    using value_type = T;
};

template<typename T>
typename T::value_type sum(T first, T last)
{
    typename iterator_traits<T>::value_type s = 0;

    while(first != last)
    {
        s = s + *first;
        ++first;
    }
}
```

단, iterator_traits는 iterator에서 제공해준다.

```cpp
template<typename T>
typename iterator_traits<T>::value_type sum(T first, T last)
{
    typename iterator_traits<T>::value_type s = 0;

    while(first != last)
    {
        s = s + *first;
        ++first;
    }
}
```

```cpp
template<typename T>
typename iterator_traits<T>::value_type sum(T first, T last)
{
    //typename iterator_traits<T>::value_type s = 0;

    typename remove_reference<decltype(*first)>::type s = 0;     // 이런식으로 처리 가능

    while(first != last)
    {
        s = s + *first;
        ++first;
    }
}
```

---

## Example

```cpp
template<typename T>
void eadvance_imp(T& p, int n, random_access_iterator_tag)
{
  cout << "rand ver." << endl;
  p = p + n;
}

template<typename T>
void eadvance_imp(T& p, int n, input_iterator_tag)
{
  cout << "input ver." << endl;
  while(n--) ++p;
}

template<typename T>
void eadvance(T& p, int n)
{
    // eadvance_imp(p, n, typename T::iterator_category());
    eadvance_imp(p, n, 
    typename iterator_traits<T>::iterator_category());
}

int main()
{
    // vector<int> s = {1,2,3,4,5,6,7,8,9,10};
    int s[10] = {1,2,3,4,5,6,7,8,9,10};

    auto p = begin(s);

    eadvance(p, 5);

    cout << "*p" << endl;
}
```