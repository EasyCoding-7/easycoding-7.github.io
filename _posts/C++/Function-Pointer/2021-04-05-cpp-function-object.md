---
layout: post
title:  "(C++ : Function-Pointer) 함수 객체"
summary: ""
author: C++
date: '2021-04-05 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['weak_ptr']
usemathjax: true
permalink: /blog/cpp/function-object/
---

함수처럼 동작하는 객체!

```cpp
void HelloWorld()
{
    cout << "Hello World" << endl;
}

int main()
{
    void (*pFunc)(void);

    pFunc = &HelloWorld
    (*pFunc)();

    // 함수 포인터 단점.
    // 1. 시그니처가 안맞으면 사용할 수 없음.(매개변수가 안맞으면 사용불가)
    // 2. 상태를 가질 수 없다.(멤버 변수를 가질수 없다.)
}
```

이런 함수 포인터의 단점을 보완해보자

```cpp
class Functor
{
public:
    void operator()()
    {
        cout << "Functor Test" << endl;
    }

    bool operator()(int num)
    {
        // 이런식으로 수정도 가능
        return true;
    }

private:
    int _value = 0;
};

int main()
{
    Functor functor;
    functor();

    return 0;
}
```

Functor의 장점이 있나??

* 함수의 호출 시점과 실행시점을 분리할 수 있다.

```cpp
class MoveTask
{
public:
    void operator()()
    {
        cout << "Move!" << endl;
    }

public:
    int _platerId;
    int _posX;
    int _posY;
};

int main()
{
    MoveTask task;
    task._playerId = 100;
    task._posX = 5;

    // 나중에 실행
    task();
}
```