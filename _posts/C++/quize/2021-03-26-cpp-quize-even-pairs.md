---
layout: post
title:  "(C++ : Quize) Even Pair"
summary: ""
author: C++
date: '2021-03-26 0:00:00 +0000'
category: ['Cpp', 'Quize']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Quize/even-pairs/
---

## Q

```
Input: "3gy41d216"
Output: true
```

```
Input: "f09r27i8e67"
Output: false
```

---

## A

```cpp
#include <iostream>
#include <string>
using namespace std;

bool is_even( string str, int i)
{
  for (int j = i; j < str.length(); j++)
  {
    if (!isdigit(str[j]))
      return false;
    int digit = str[j]-'0';
    if (digit % 2 == 0)
      return true;
  }
    return false;
}
string EvenPairs(string str) {
  // code goes here 
   for (int i = 0; i < str.length(); i++)
  {
    if (!isdigit(str[i]))
      continue;
    int digit = str[i] - '0';
    if (digit % 2 != 0)
      continue;
    if (is_even(str, i+1))
      return "true";
   }
    return "false";

}

int main(void) { 
   
  // keep this function call here
  cout << EvenPairs(coderbyteInternalStdinFunction(stdin));
  return 0;
    
}
```