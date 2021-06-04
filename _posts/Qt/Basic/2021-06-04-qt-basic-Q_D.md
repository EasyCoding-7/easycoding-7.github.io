---
layout: post
title:  "(Qt : basic) Q_D Macro"
summary: ""
author: Qt
date: '2021-06-04 0:00:00 +0000'
category: ['Qt']
#tags: ['Qt', 'tag1-test1']
thumbnail: /assets/img/posts/qt-thumnail.png
#keywords: ['Qt 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Qt/basic/q-d-macro/
---

* [참고사이트1](https://wiki.qt.io/D-Pointer)
* [참고사이트2](https://idlecomputer.tistory.com/105)
* [참고사이트3](https://pythonq.com/so/c%2B%2B/662617)

---

우선 이걸 먼저 알아야한다.

```cpp
// Myclass.h
class Myclass
{
public:
    Myclass() {}
    ~Myclass() {}

    // some func

private:
    int m_data1;
};
```

처음에 이러한 `Myclass`클래스가 있었다 가정해보자.<br>
사용중 어떠한 이유로 멤버 데이터를 확장해야할 일이 생겼다.

```cpp
// Myclass.h
class Myclass
{
public:
    Myclass() {}
    ~Myclass() {}

    // some func

private:
    int m_data1;
    std::string m_string1;
    // ... (엄청 많다고 가정)
};
```

이게 효율적일까?<br>
당연하지만 비효율적 -> `Myclass.h`를 참조하는 모든 cpp에서 컴파일시 수정이 필요

개선하는 방법은 간단하다. 멤버변수를 다른곳에 옮겨두면된다.

```cpp
class Myclass
{
public:
    Myclass() : d_ptr(new MyclassPrivate) {}
    ~Myclass() {}

    // some func
protected:
    MyclassPrivate *d_ptr;
};
```

```cpp
struct MyclassPrivate
{
    int m_data1;
    std::string m_string1;
    // ... (엄청 많다고 가정)
};
```

예를들어서 `m_data1`에 접근한다면

```cpp
class Myclass
{
public:
    Myclass() : d_ptr(new MyclassPrivate) {}
    ~Myclass() {}

    // 이렇게 접근가능
    int getMydata1() { return d_ptr->m_data1; }

    // some func
protected:
    MyclassPrivate *d_ptr;
};
```

---

멀리왔다... 그래서 `Q_D, Q_Q`를 왜쓰는데?

선언은 아래와 같이 되어있다.

```cpp
#define Q_D(Class) Class##Private * const d = d_func()
#define Q_Q(Class) Class * const q = q_func()
```

문제는 뭐냐? d_ptr를 쓰는 클래스가 d_ptr를 쓰는 부모클래스를 상속해버리는 경우가 발생한다.

```cpp
class MyclassBase
{
public:
    MyclassBase() : d_ptr(new MyclassBasePrivate) {}
    ~MyclassBase() {}

    // some func
protected:
    MyclassBasePrivate *d_ptr;
};
```

```cpp
class MyclassDerived
{
public:
    MyclassDerived() : d_ptr(new MyclassDerivedPrivate) {}
    ~MyclassDerived() {}

    int getMyData() { 
        // 컴파일러 입장에서 d_ptr가 MyclassDerived인지 MyclassBase인지 구분 불가능
        return d_ptr->mydata; 
        }

    // some func
protected:
    MyclassDerivedPrivate *d_ptr;
};
```

캐스팅을 새로할까?

```cpp
int getMyData() 
{ 
    // 매번 이걸한다고?? -> Q_D, Q_Q를 사용하자
    MyclassDerivedPrivate* d = static_cast<MyclassDerivedPrivate>(d_ptr);
    return d->mydata; 
}
```

---

```cpp
class MyclassDerived
{
    Q_DECLARE_PRIVATE(MyclassDerived)   // Q_Q를 쓰기위해서 선언

public:
    MyclassDerived() : d_ptr(new MyclassDerivedPrivate) {}
    ~MyclassDerived() {}

    int getMyData() 
    { 
        // good!
        Q_D(MyclassDerived)
        return d->mydata; 
    }

    // some func
protected:
    MyclassDerivedPrivate *d_ptr;
};
```