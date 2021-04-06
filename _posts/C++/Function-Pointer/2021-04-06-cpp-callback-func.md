---
layout: post
title:  "(C++ : Function-Pointer) 콜백 함수"
summary: ""
author: C++
date: '2021-04-06 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['weak_ptr']
usemathjax: true
permalink: /blog/cpp/callback-func/
---

```cpp
class Item
{
public:

public:
    int _itemId = 0;
    int _rarity = 0;
    int _ownerId = 0;
};

class FindByOwnerId
{
public:
    bool operator()(const Item* item)
    {
        return (item->_ownerId == _ownerId);
    }

public:
    int _ownerId;
};

class FindByRarity
{
public:
    bool operator()(const Item* item)
    {
        return (item->_rarity == _rarity);
    }

public:
    int _rarity;
};

Item* FindItem(Item items[], int itemCount, /* Functor를 넘겨주고 싶다 */)
{
    for(int i = 0; i < itemCount; i++)
    {
        Item* item = &items[i];

        return item;
    }
}

int main()
{
    Item items[10];

    FindItem(items, 10, /*Functor*/);
}
```

```cpp
template<typename T>
Item* FindItem(Item items[], int itemCount, T selector)
{
    for(int i = 0; i < itemCount; i++)
    {
        Item* item = &items[i];

        if(selector(item))
            return item;
    }
}

int main()
{
    Item items[10];

    FindByOwnerId functor1;
    functor1._owner = 10;

    FindByRarity functor2;
    functor2._rarity = 3;

    Item* i = FindItem(items, 10, functor1);
    Item* i2 = FindItem(items, 10, functor2);
}
```

