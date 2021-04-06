---
layout: post
title:  "(C++ : Function-Pointer) Template"
summary: ""
author: C++
date: '2021-04-05 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['weak_ptr']
usemathjax: true
permalink: /blog/cpp/template/
---

## 템플릿

함수나 클래스를 찍어내는 틀

```cpp
void Print(int a)
{
    cout << a << endl;
}

int main()
{
    Print(50);  // 정수말고 다른걸 출력하고 싶다면??
    // 매번 함수를 새로 생성해야할까?? -> 함수 템플릿!!
}
```

```cpp
template<typename T>
void Print(T a)
{
    cout << a << endl;
}
```

템플릿 특수화

```cpp
template<>
void Print(Knight a)        // 특정 변수/객체 에 대해 반응
{
    // ...
}
```

---

## 클래스 템플릿

```cpp
class RandomBox
{
public:
    int GetRandomData()
    {
        int idx = rand() % 10;
        return _data[idx];
    }

public:
    int _data[10];
}

int main()
{
    stand(static_cast<unsinged int>(time(nullptr));

    RandomBox rb1;
    RandomBox rb2;

    float f = rb1.GetRandomData();  // float으로 받을 순 없나??
    // 템플릿을 써보자
}
```

```cpp
template<typename T>
class RandomBox
{
public:
    T GetRandomData()
    {
        // ...
    }
}

// ...

    RandomBox<int> rb1;
    RandomBox<float> rb2;
```

이런 선언도 가능하다

```cpp
template<typename T, int SIZE = 10>
class RandomBox
{
public:
    T GetRandomData()
    {
        int idx = rand() % 10;
        return _data[idx];
    }

public:
    T _data[SIZE];
}

// ...

    RandomBox<int> rb1;         // ok
    RandomBox<float, 20> rb2;   // ok
    RandomBox<float, 30> rb3;   // ok, 단 rb2와는 완전히 다른 새로운 클래스가 생성되는 개념임

    rb2 = rb3;       // error - 완전다른 타임이라 캐스팅 불가능
```
