---
layout: post
title:  "(C++) 헷갈리는 부분 정리"
summary: ""
author: C++
date: '2021-05-24 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/Etc/
---

## #define문에서 ##

```cpp
#define ArgArg(x, y)          x##y
#define ArgText(x)            x##TEXT
#define TextArg(x)            TEXT##x
#define TextText              TEXT##text
#define Jitter                1
#define bug                   2
#define Jitterbug             3
```

결과출력

```
ArgArg(lady, bug)       "ladybug"
ArgText(con)            "conTEXT"
TextArg(book)           "TEXTbook"
TextText                "TEXTtext"
ArgArg(Jitter, bug)     3
```

---

## mutable

```cpp
class Point
{
  int x, y;

  // mutable 선언
  mutable int cnt = 0;

public:
    Point(int a = 0, int b = 0) : x(a), y(b) {}

    void print() const
    {
        ++ cnt;   // 상수함수인데 변경가능.
        std::cout << x << ", " << y << std::endl;
    }
};
```

---

## friend 함수

멤버 함수는 아니지만 private 멤버에 접근 할 수 있게 하고 싶을 때 사용<br>
특정 함수, 클래스에서 접근이 가능하게 하고 싶을때.<br>

```cpp
class Airplane
{
    int color;
    int speed;
    int engineTemp;
  
public:
    int getSpeed() { return speed; }
    // void fixAirplane(Airplane& a);
    friend void fixAirplane(Airplane& a);   // 내부 멤버에 접근가능!
};

void fixAirplane(Airplane& a)
{
    int n = a.engineTemp;   // engineTemp 접근 불가
}
```

---

## volatile

변수를 선언할 때 앞에 volatile을 붙이면 컴파일러는 해당 변수를 최적화에서 제외하여 항상 메모리에 접근하도록 만듭니다.

```cpp
int i = 0;

while (i < 10)
    i++;

printf("%d\n", i);    // 10
```

volatile을 쓰지않으면 위의 코드는 컴파일러에 의해서

```cpp
int i = 10;    // 반복문을 없애버리고 10을 할당

printf("%d\n", i);    // 10
```

이렇게 된다.

```cpp
volatile int i = 0;    // volatile로 선언하여 항상 메모리에 접근하도록 만듦

// 항상 i의 메모리에 접근해야 하므로 컴파일러는 반복문을 없애지 않음
while (i < 10)
    i++;

printf("%d\n", i);    // 10
```

---

## const와 constexpr

```cpp
// constexpr은 컴파일 시간 상수 만 넣을 수 있다. (컴파일 할 당시 상수인 것)
int n = 10;

const int c1 = 10;    // 컴파일 시간에 알 수 있다.(컴파일 시간 상수)
const int c2 = n;     // 컴파일 시간에 알 수 없다. (실행시간 상수)

constexpr int c3 = 10;    // Ok
constexpr int c4 = n;     // Compile Error, 컴파일 시간 상수만 넣을 수 있다.
```

---

## extern

```cpp
// B.cpp
#include <stdio.h>

int num1 = 10;      // 여기서 선언

void printNumber()
{
    print("%d\n", num1);
}
```

```cpp
// A.cpp
#include <stdio.h>

extern int num1;        // 여기서 사용

int main()
{
    print("%d\n", num1);
    return 0;
}
```

---

## #include <stdio.h> / #include <cstdio> 차이

* `#include <stdio.h>` - c표준, printf등이 global 네임스페이스 밑에 있음.
* `#include <cstdio>` - cpp표준, printf등이 std네임스페이스 밑에 있음

---

## iomanipulator

```cpp
#include <iostream>

int main()
{
    int n = 10;
    std::cout << n << std::endl;              // 10 진수
    std::cout << std::hex << n << std::endl;  // 16 진수
    std::cout << n << std::endl;              // 10 진수??
    // 16진수로 출력되게 된다.

    std::cout << std::dec;      // 다시 10진수로 변경
}
```

* `std::dec` - 변수값을 10진수로 출력
* `std::hex` - 변수값을 16진수로 출력
* `std::setw` - 문자열 출력시 개수 지정
* `std::setfill` -공백을 채울 문자 지정
* `std::left` - 왼쪽 정렬