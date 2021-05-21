---
layout: post
title:  "(C++ : Move Semantics-1) 컴파일러가 제공해주는 복사생성자 모양"
summary: ""
author: C++
date: '2021-05-21 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/move-semantics/copy-constructor/
---

컴파일러가 제공해주는 복사생성자는 어떻게 생겼을까?

```cpp
class Point
{
  int x, y;
public:
  Point(int a = 0, int b = 0) : x(a), y(b) {}
  
  Point(const Point& p)   // 컴파일러가 제공해주는 모양
  {
    // 모든 멤버 복사
  }
};

int main()
{
  Point p1(1, 1);

  // Point(Point) - 복사 생성자가 필요하다., 우선적으로 컴파일러가 제공은해준다.
  Point p2(p1);
}
```

```cpp
// 그런데 복사 생성자를 이런모양으로 만들면 안되나?
Point(Point p)
// 이 모양의 문제는 Point p = p1의 꼴을 갖기에 무한히 복사 생성자를 호출하게 된다.
// ** 복사 생성자는 Call-by-value를 사용할 수 없다. **
```

```cpp
// 그럼 이 모양은?
Point(Point& p)
{

} // 여기까진 문제가 없음

Point foo()
{
  Point ret(0, 0);
  return ret;
}

int main()
{
    Point p3(foo());
    // foo()의 결과는 임시객체로 현재 복사 생성자는 임시객체를 받을 수 없다.
}

// l, r value를 모두 받을수 있는 방법은
Point(const Point& p) 
```