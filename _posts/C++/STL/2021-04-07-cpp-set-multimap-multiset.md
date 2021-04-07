---
layout: post
title:  "(C++) Set, MultiMap, MultiSet"
summary: ""
author: C++
date: '2021-04-07 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['list']
usemathjax: false
permalink: /blog/cpp/set-multimap-multiset/
---

## set

* map은 key, value형태
* set은 value가 key인 형태

```cpp
set<int> s;

s.insert(10);
s.insert(20);
s.insert(30);
s.insert(40);
s.insert(50);

s.erase(10);
s.erase(30);
s.erase(50);

set<int>::iterator findit = s.find(40);
if(findit == s.end())
{
    // 못찾음
}

for(set<int>::iterator it = s.begin(); it != s.end(); ++it)
{
    cout << (*it) << endl;
}
```

---

## multimap

* 중복 key를 허용

```cpp
multimap<int, int> mm;

mm.insert(make_pair(1, 100));
mm.insert(make_pair(1, 200));
mm.insert(make_pair(2, 100));
mm.insert(make_pair(2, 200));

// mm[1] = 500;    // 막혀있음.

mm.erase(1);        // key가 여러개더라도 다 삭제됨
// (2, 100) (2, 200) 만 남음

unsigned int count = mm.erase(1);

multimap<int, int>::iterator itFind = mm.find(1);
if(itFind != mm.end())
{
    // 찾음
}

pair<multimap<int, int>::iterator, multimap<int, int>::iterator> itPair;
itPair = mm.equal_range(1);
// 이렇게도 쓸 수 있음.
// multimap<int, int>::iterator itBegin = mm.lower_bound(1);
// multimap<int, int>::iterator itEnd = mm.upper_bound(1);
for(multimap<int, int>::iterator it = itPair.first; it != itPair.second; ++it)
{
    cout << it->first << " " << it->second << endl;
}
```

---

## multiset

```cpp
multiset<int> ms;

ms.insert(100);
ms.insert(100);
ms.insert(100);
ms.insert(200);
ms.insert(200);

multiset<int>::iterator findit = ms.find(100);

pair<multiset<int, int>::iterator, multiset<int, int>::iterator> itPair;
itPair = ms.equal_range(100);
for(multiset<int, int>::iterator it = itPair.first; it != itPair.second; ++it)
{
    cout << it->first << " " << it->second << endl;
}
```

---

## Example

