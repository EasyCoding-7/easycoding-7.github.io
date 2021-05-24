---
layout: post
title:  "(Modern C++ : 11~14) foreach"
summary: ""
author: C++
date: '2021-05-24 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/cpp/modern-cpp-11-14/foreach/
---

```cpp
// C++ 03
for(vector<int>::iterator itr = v.begin(); itr != v.end(); ++itr)
    cout << (*itr);

// C++ 11
for(int i: v)
    cout << i;
```