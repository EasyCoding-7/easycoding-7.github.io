---
layout: post
title:  "(C++) unique_ptr"
summary: ""
author: C++
date: '2021-03-25 0:00:00 +0530'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/unique_ptr/
---

## unique_ptr이란?

* 메모리주소의 참조를 독점할 수 있는 포인터
* 왜쓰나? 굳이 소멸자를 호출하지 않아도 되는 장점이 있다!

```cpp
#include <iostream>
#include <memory>

using namespace std;

// Data structure representing a point on the screen
struct point {
    int x;
    int y;
};

int main() {
	// Create a unique_ptr to an point which has initial value {3,6}
	auto p{ make_unique<point>( point{3, 6} ) };
	//unique_ptr<point> p{ new point{3, 6} };          // C++11

	cout << p->x << ", " << p->y << endl;
}
```

```cpp
#include <iostream>
#include <memory>

using namespace std;

// Data structure representing a point on the screen
struct point {
    int x;
    int y;
};

unique_ptr<point> create_ptr() { 
    point p{0};                                  // Create point instance
    p.x = 3;                                     // Populate point instance
	p.y = 6;	
    unique_ptr<point> ptr = make_unique<point>(p);     // Create local unique_ptr instance
    return ptr;                                    // The pointer member is transferred
                                                 // from p to the returned instance
}

int main() {
	unique_ptr<point> upp { create_ptr() };

	cout << upp->x << ", " << upp->y << endl;
}
```

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Car
{
    int color;
    int speed;
public:
    ~Car() { cout << "~Car()" << endl; }
 
    void Go() { cout << "Car go" << endl; }
};

int main()
{
    shared_ptr<Car> sp1(new Car);
    shared_ptr<Car> sp2 = sp1;      // 참조 계수 2

    unique_ptr<Car> up1(new Car);   // 자원 독점
    unique_ptr<Car> up2 = up1;      // error
```

## 그런데 그래도 옮겨야할 때가 발생한다면?? (move이용)

```cpp
#include <iostream>
#include <memory>
using namespace std;

int main()
{
    unique_ptr<int> up1(new int);

    unique_ptr<int> up2 = move(up1);        // ok.
}
```

## 삭제자 선언하기

```cpp
#include <iostream>
#include <memory>
using namespace std;

struct Deleter
{
    void operator()(int* p) const
    {
        delete p;
    }
}

int main()
{
    unique_ptr<int> up1(new int);

    // 삭제자는 이렇게 선언
    unique_ptr<int, Deleter> up(new int);

    // 이런 삭제자도 가능 - 1
    unique_ptr<int, void(*)(int*)> up(new int, foo);

    // 이런 삭제자도 가능 - 2
    auto f = [](int*p){delete p;}
    unique_ptr<int, decltype(f)> up(new int, f);
}
```

```cpp
// 배열도 사용가능 (C++11)
unique_ptr<int[]> up(new int[10]);
```

## shared_ptr <-> unique_ptr 호환성

```cpp
#include <iostream>
#include <memory>
using namespace std;

int main()
{
    shared_ptr<int> sp(new int);
    unique_ptr<int> up(new int);

    shared_ptr<int> sp1 = up;       // error
    shared_ptr<int> sp2 = move(up); // ok

    unique_ptr<int> up1 = sp;       // error
    unique_ptr<int> up2 = move(sp); // error
}
```

---

## Example

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Shape {};
class Rect : public Shape {};
class Circle : public Shape {};

Shape* CreateShape(int type)
{
    Shape* p = nullptr;

    switch(type)
    {
        case 1 : p = new Rect; break;
        case 2 : p = new Circle; break;
    }
    return p;
}

int main()
{
    Shape* p = CreateShape(1);
```

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Shape {};
class Rect : public Shape {};
class Circle : public Shape {};

unique_ptr<Shape> CreateShape(int type)
{
    unique_ptr<Shape> p;

    switch(type)
    {
        case 1 : p.reset(new Rect); break;
        case 2 : p.reset(new Circle); break;
    }
    return p;
}

int main()
{
    // unique_ptr<Shape> p = CreateShape(1);
    shared_ptr<Shape> p = CreateShape(1);
    // unique, shared 모두 사용가능
```

```cpp
unique_ptr<uint8_t[]> mp_array;

// 선언
mp_array = std::make_unique<uint8_t[]>(48000 * 8);

// uint8_t* 의 호출이 필요할 경우
&mp_array[0];
```