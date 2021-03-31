---
layout: post
title:  "(C++ : Design-pattern) Upcasting"
summary: ""
author: C++
date: '2021-03-31 0:00:00 +0000'
category: ['Cpp', 'Design-pattern']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/design-pattern/upcasting/
---

## Upcasting이란

상속을 동족끼리 묶을때 사용한다는 개념으로 사용된다.

```cpp
class Animal
{
    int age;
};

class Dog : public Animal
{
    int color;
};

int main()
{
    Dog d;
    Dog* p1 = &d;           // ok
    double* p2 = &d;        // error
    Animal* p3 = &d;        // ok - upcasting
}
```

## 주의할 점은?

```cpp
class Animal
{
    int age;
public:
    void Cry() { cout << "Animal Cry" << endl; }
};

class Dog : public Animal
{
    int color;
public:
    // override
    void Cry() { cout << "Dog Cry" << endl; }
};

int main()
{
    Dog d;
    Animal* p = &d;

    p->Cry();   // "Animal Cry" -> 포인터를 보고 따라간다.
    // 단, 자바와 C#에서는 Dog를 부름
}
```

```cpp
// 만약 Dog Cry를 부르고 싶다면?
class Animal
{
    int age;
public:
    virtual void Cry() { cout << "Animal Cry" << endl; }
};

class Dog : public Animal
{
    int color;
public:
    // override
    virtual void Cry() override { cout << "Dog Cry" << endl; }
};
```