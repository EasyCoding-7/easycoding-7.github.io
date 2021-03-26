---
layout: post
title:  "(C++ : Quize) Off Binary"
summary: ""
author: C++
date: '2021-03-24 0:00:00 +0000'
category: ['Cpp', 'Quize']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Quize/off-binary/
---

## Q

```
Input: {"5624", "0010111111001"}
Output: 2
```

```
Input: {"44", "111111"}
Output: 3
```

---

## A

```cpp
#include <iostream>
#include <string>
using namespace std;

string OffBinary(string strArr[], int arrLength) {
  
  // code goes here
  int d = stol(strArr[0]);
  int count = 0;
  for(size_t i=0;i<strArr[1].length();i++)
  {
    int decimal = d&1;      // 현재 자리가 0인지 1인지 판별
    d = d>>1;               // 나누기 2를 해준다.
    int binary = strArr[1][strArr[1].length()-1-i] - '0';   // 비교할 문자확인
    if(decimal!=binary)
      count++;    
  }

  /*
    2진수 만드는 법을 알아야 함
    5 -> 홀수면 1, 짝수면 0 -> 나누기 2
    5 -> 홀수 1
    2 -> 짝수 0
    1
  */
   
  return to_string(count);

}

int main(void) { 
   
  // keep this function call here
  string A[] = coderbyteInternalStdinFunction(stdin);
  int arrLength = sizeof(A) / sizeof(*A);
  cout << OffBinary(A, arrLength);
  return 0;
    
}
```