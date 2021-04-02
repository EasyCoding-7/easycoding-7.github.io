---
layout: post
title:  "(C++ : Design-pattern) Adapter Pattern"
summary: ""
author: C++
date: '2021-04-02 0:00:00 +0000'
category: ['Cpp', 'Design-pattern']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/design-pattern/adapter-pattern/
---

그림판에서 아래와 같이 텍스트를 관리하고 있었다고 가정해보자.

```cpp
// TextView.h
#include <string>
#include <iostream>

class TextView
{
    std::string data;
    std::string font;
    int width;
public:
    TextView(std::string s, std::string fo = "나눔고딕", int w = 24) : data(s), font(fo), width(w) {}
    void Show() { std::cout << data << std:: endl; }
};
```

그럼 도형은 어떻게 관리하고 있었을까?

```cpp
#include <iostream>
#include <vector>
using namespace std;

class Shape
{
public:
    virtual void Draw() { cout << "Draw Shape" << endl; }
};

class Rect : public Shape
{
public:
    virtual void Draw() { cout << "Draw Rect" << endl; }
};

class Circle : public Shape
{
public:
    virtual void Draw() { cout << "Circle Rect" << endl; }
};

int main()
{
    vector<Shape*> v;
    v.push_back(new Rect);
    v.push_back(new Circle);

    for( auto p : v)
        p->Draw();
}
```

기능이 추가되어 텍스트를 마치 도형처럼 편집하고자 한다면?(`vector<Shape*>`에 TextView를 넣고싶다면?)

```cpp
class Text : public TextView, public Shape
{
public:
    Text(string s) : TextView(s) {}
    virtual void Draw() { TextView::Show(); }
};
```

```cpp
int main()
{
    vector<Shape*> v;
    v.push_back(new Rect);
    v.push_back(new Circle);
    v.push_back(new Text("hello"));

    for( auto p : v)
        p->Draw();
}
```

이런것을 Adapter Pattern이라 한다.

## Adapter 패턴

* 한 클래스의 인터페이스를 클라이언트가 사용하고자 하는 다른 인터페이스로 변환
* 호환성 때문에 사용할 수 없었던 클래스들을 연결해서 사용할 수 있다.

---

## object adapter(객체 어댑터)

```cpp
class Text : public TextView, public Shape
{
public:
    Text(string s) : TextView(s) {}
    virtual void Draw() { TextView::Show(); }
};

int main()
{
    vector<Shape*> v;

    TextView tv("world");
    v.push_back(&tv);  
    // error! - TextView라는 객체 자체를 어댑팅할 수 있게 만들어야한다. -> 객체 어답터
    v.push_back(new Text("hello"));

    for( auto p : v)
        p->Draw();
}
```

```cpp
class ObjectAdapter : public public Shape
{
    TextView* pView;
public:
    ObjectAdapter(TextView* p) : pView(p) {}
    virtual void Draw() { pView->Show(); }
};

int main()
{
    vector<Shape*> v;

    TextView tv("world");
    v.push_back(new ObjectAdapter(&tv));    // ok, 객체 어답터라 한다.
}
```

* 클래스 어답터
    * 위에서 구현한 Text는 일종의 클래스 어답터이다.
    * 클래스 인터페이스를 변경
    * 다중 상속 또는 값으로 포함 하는 경우
    * 이미 존재하던 객체의 인터페이스는 변경할 수 없다.
* 객체 어답터
    * 객체의 인터페이스를 변경
    * 구성(Composition)을 사용하는 경우가 많다
    * 기존 객체를 포인터 또는 참조로 
    
---

## STL에서는 adapter를 어떻게 쓸까?

```cpp
#include <iostream>
#include <list>
using namespace std;

// list를 이용하여 stack을 만들어 보자.
// list의 함수이름을 stack 처럼 보이도록 변경하자.
template<typename T> class Stack : public list<T>
{
public:
    void push(const T& a) { list<T>::push_back(a); }
    void pop()            { list<T>::pop_back(a); }
    T& top()              { return list<T>::back(); }
};

int main()
{
    Stack<int> s;
    s.push(10);
    s.push(20);

    s.push_front(20);       
    // 문제) stack이니 이런걸 막아야할 텐데?

    cout << s.top() << endl;
}
```

```cpp
// 방법 1. private 상속
// 함부로 list에 접근하지 못하게 막음
template<typename T> class Stack : private list<T>
```

```cpp
// 방법 2. 멤버 객체로 받는다.
template<typename T> class Stack : public list<T>
{
    list<T> st;
public:
    void push(const T& a) { st.push_back(a); }
    void pop()            { st.pop_back(a); }
    T& top()              { return st.back(); }
```

```cpp
// 방법 2를 조금 더 진화
template<typename T, typename C = deque<T>> class Stack : public list<T>
{
    C st;
    // ...
```

사실 이런게 C++표준에 다 있다.

```cpp
#include <stack>

int main()
{
    stack<int> s;
    s.push(10);
    s.push(20);
}
```

이 stack도 클래스 어뎁터이다.

---

## Example

```cpp
#include <iostream>
#include <algorithm>
#include <list>
using namespace std;

int main()
{
    list<int> s = { 1, 2, 3, 4 };

    auto p1 = s.begin();
    auto p2 = s.end();

    for_each(p1, p2, [](int a) { cout << a << endl; });
    // 꺼꾸로 출력하고 싶다면??
}
```

```cpp
#include <iostream>
#include <algorithm>
#include <list>
using namespace std;

int main()
{
    list<int> s = { 1, 2, 3, 4 };

    auto p1 = s.begin();
    auto p2 = s.end();

    reverse_iterator<list<int>::iterator> p3(p2);
    reverse_iterator<list<int>::iterator> p4(p1);

    // 위와 동일한 표현
    auto p3 = make_reverse_iterator(p2);

    for_each(p3, p4, [](int a) { cout << a << endl; });
}
```