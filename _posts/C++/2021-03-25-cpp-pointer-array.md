---
layout: post
title:  "(C++) Pointer Array"
summary: ""
author: C++
date: '2021-03-25 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
keywords: ['pointer array']
usemathjax: true
permalink: /blog/cpp/pointer-array/
---

```cpp
#include <iostream>

using namespace std;

int main() {
	int *pa = new int[20];
	
	for (int i = 0; i < 20; ++i) {
		pa[i] = i;
	}
	
	for (int i = 0; i < 20; ++i) {
		cout << pa[i] << ", ";
	}
	cout << endl;
	
	delete [] pa;
}
```