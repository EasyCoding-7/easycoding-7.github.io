---
layout: post
title:  "(C++ STL : iterator-3) advance function(dispathcing 써보기)"
summary: ""
author: C++
date: '2021-05-23 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/stl/iterator-3/
---

```cpp
#include <iostream>
#include <vector>
#include <list>
#include <iterator>
using namespace std;

int main()
{
    list<int> s = {1,2,3,4,5,6,7,8,9,10};
    auto p = begin(s);

    advance(p, 5);    // p+5를 의미한다.
    cout << *p << endl;
}
```

## advance만들어 보기

```cpp
template<typename T>
void eadvance(T& p, int n)
{
  while(n--)
    ++p;      // 이런식으로 모든 컨테이너에 적용이 가능하지만, 
              // 만들면 성능의 저하가 크다
}
```

```cpp
struct input_iterator_tag {};   // 아무런 멤버가 없는 empty class
struct output_iterator_tag {};

struct forward_iterator_tag : public input_iterator_tag {};
struct bidirectional_iterator_tag : public forward_iterator_tag {};
struct random_access_iterator_tag : public bidirectional_iterator_tag {};
```

위를 카테고리 테그라 하며 algorithm.h에 정의 되어 있다.

```cpp
template<typename T> class vector_iterator
{
public:
  using iterator_categoty = random_access_iterator_tag;
};

template<typename T> class list_iterator
{
public:
  using iterator_categoty = bidirectional_iterator_tag;
};
```

어떤 종류의 반복자인지 확인가능<br>
이제 사용해보자

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
  // 반복자의 종류에 따라
  eadvance_imp(p, n, typename T::iterator_category());
}
```

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
  // dispathcing 방벙 1 - 함수 인자를 이용한 오버로딩
  eadvance_imp(p, n, typename T::iterator_category());

  // dispathcing 방벙 2 - if문 사용(C++17)
  if constexpr (is_same<typename T::iterator_category, random_access_iterator_tag>::value)
  {
    p = p + n;
  }
  else
    while(n--) ++p;
}

// dispathcing 방벙 3 - enable_if 사용 : SFINAE 사용
template<typename T>
typename enable_if<is_same<typename T::iterator_category, random_access_iterator_tag>::value>::type eadvance(T& p, int n)
{
  p = p + n;
}

template<typename T>
typename enable_if<is_same<typename T::iterator_category, input_iterator_tag>::value>::type eadvance(T& p, int n)
{
  while(n--) ++p;
}
```