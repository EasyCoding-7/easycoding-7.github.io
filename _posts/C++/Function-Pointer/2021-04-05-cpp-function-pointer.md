---
layout: post
title:  "(C++ : Function-Pointer) 함수 포인터"
summary: ""
author: C++
date: '2021-04-05 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['weak_ptr']
usemathjax: true
permalink: /blog/cpp/function-pointer/
---

```cpp
int Add(int a, int b)
{
    return a + b;
}

// 함수 타입정의
typedef int(FUNC_TYPE)(int a, int b);
// using FUNC_TYPE = int(int a, int b);     // 동일한 표현

FUNC_TYPE* fn;      // 이런식의 사용이 가능할까?

int result = Add(1, 2);
cout << result << endl;    // 3

fn = Add;
result = fn(3, 4);
// result = (*fn)(3, 4);    // 동일한 표현이다.(함수 포인터이기에 명시적으로 알려줄수있음.)
cout << result << endl;     // 7
```

근데 이게 왜 필요해??<br>
만약 마이너스 함수가 추가되었다고 가정해보자.

```cpp
int Add(int a, int b)
{
    return a + b;
}

int Sub(int a, int b)
{
    return a - b;
}

typedef int(FUNC_TYPE)(int a, int b);
FUNC_TYPE* fn;

fn = Add;
cout << fn(3, 2) << endl;       // 5

fn = Sub;
cout << fn(3, 2) << endl;       // 1

// 만약 함수포인터를 쓰지 않았다면? 
// Sub를 하나하나 Add로 변경해야할 것이다.
```

아니 아무리 그래도 ... 이걸 써야한다고??

```cpp
class Item
{
public:
    Item() : _itemId(0), _rarity(0), _ownerId(0)
    {

    }

public:
    int _itemId;
    int _rarity;
    int _ownerId;
};

Item* FindItem(Item items[], int itemCount, int itemId)
{
    for(int i = 0; i < itemCount; i++)
    {
        Item* item = &items[i];
        // TODO 조건
        if(item->_itemId == itemId)
        {
            // 매번 이걸 반복해야 할까??
            return item;
        }
    }

    return nullptr;
}
```

FindItem를 재활용할 방안을 생각해 보자

```cpp
// 요렇게 해보자.
Item* FindItem(Item items[], int itemCount, /* 동작 */)
{
    for(int i = 0; i < itemCount; i++)
    {
        Item* item = &items[i];
        // TODO 조건
        /* 동작 */
    }

    return nullptr;
}
```

```cpp
bool IsRareItem(Item* item)
{
    return item->_rarity >= 2;
}

typedef bool(ITEM_SELECTOR)(Item* item);

// Item* FindItem(Item items[], int itemCount, bool(*selector)(Item* item))
// 좀 더 멋지게 표현
Item* FindItem(Item items[], int itemCount, ITEM_SELECTOR* selector)
{
    for(int i = 0; i < itemCount; i++)
    {
        Item* item = &items[i];
        
        if(selector(item)) return item;
    }

    return nullptr;
}

int main()
{
    item items[10] = {};
    item[3]._rarity = 3;
    item* rareItem = FindItem(items, 10, isRareItem);
}
```

---

```cpp
int Test(int a, int b)
{
    cout << "Test" << endl;
    return a + b;
}

int main()
{
    int (*fn)(int, int);

    fn = &Test;
    // fn = Test;   // 이렇게 선언해도 동일함.

    fn(1, 2);
    // (*fn)(1, 2); // 역시 이렇게 선언 가능

    return 0;
}
```

단, 현재까지 만듬 함수 포인터는 전역/정적(static)함수만 담을 수 있다.

```cpp
class Knight
{
public:
    int GetHp() { return _hp; }

    int _hp{};
};

typedef int(*PFUNC)();

int main()
{
    PFUNC fn;
    fn = &Knight::GetHp;    // error!
}
```

어떻게 수정해야 할까?

```cpp
typedef int(*PFUNC)();              // 일반 함수 포인터
typedef int(Knight::*MEM_PFUNC)();      // 멤버 함수 포인터

int main()
{
    MEM_PFUNC fn;
    fn = &Knight::GetHp;    // ok - 단, 멤버 함수 포인터는 & 생략 불가

    Knight k1;
    (k1.*fn)(1,1);  // 이렇게 써야함.
}
```