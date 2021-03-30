---
layout: post
title:  "(C++ : Quize) Look Say Sequence"
summary: ""
author: C++
date: '2021-03-30 0:00:00 +0000'
category: ['Cpp', 'Quize']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Quize/look-say-sequence/
---

## Q

```
Input: 1211
Output: 111221
# 한개의 일, 한개의 이, 두개의 일
```

```
Input: 2466
Output: 121426
```

---

## A

* [Test Link](https://ideone.com/)

```cpp
#include <iostream>
#include <string>
using namespace std;

int LookSaySequence(int num) {
  
  // code goes here  
  
  return num;
}

int main(void) { 
   
  // keep this function call here
  cout << LookSaySequence(1211);
  return 0;
    
}
```

```cpp
    for(size_t j=i+1;j<number.length();j++)
    {
        // 같은 숫자일 경우 pos++
      if(number[i]==number[j])
      {
        pos++;
      }
      else
      {
        pos = 1;
        break;
      }
    }
```

---

## A - Code

```cpp
#include <iostream>
#include <string>
using namespace std;

int LookSaySequence(int num) {
  
  // code goes here  
  string number;
  number = to_string(num);
  string ret;

  size_t pos = 1;
  for(size_t i=0;i<number.length();i=i+pos)
  {
    string n;
    n = number[i];
  
    for(size_t j=i+1;j<number.length();j++)
    {
      if(number[i]==number[j])
      {
        pos++;
      }
      else
      {
        pos = 1;
        break;
      }
    }
    ret += to_string(pos)+n;
  }


  return stoi(ret);

}

int main(void) { 
   
  // keep this function call here
  cout << LookSaySequence(coderbyteInternalStdinFunction(stdin));
  return 0;
    
}
```