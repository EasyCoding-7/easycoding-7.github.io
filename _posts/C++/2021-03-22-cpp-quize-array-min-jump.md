---
layout: post
title:  "(C++ : Quize) Array Min Jumps"
summary: ""
author: C++
date: '2021-03-22 0:00:00 +0000'
category: ['Cpp', 'Quize']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Quize/array-min-jumps/
---

## Q

0번 배열에서 시작해서 해당 배열만큼 이동가능, 최소로 점프해서 이동가능한 점프수 체크

```
Input: {3, 4, 2, 1, 1, 100}
Output: 2
```

```
Input: {1, 3, 6, 8, 2, 7, 1, 2, 1, 2, 6, 1, 2, 1, 2}
Output: 4
```

---

## A

```cpp
#include <iostream>
#include <string>
using namespace std;

int ArrayMinJumps(int arr[], int arrLength) {
  int here = 0;         // 현재위치
  int there = 0;        // 내가 갈 곳
  int next_pot = 1;     // 내가 갈 곳의 길이?

  int jumps = 0;

  while (here < arrLength-1) {
    for (int next = here+1; next <= here+arr[here]; ++next) {   // 내가 갈 수 있는 곳중 가장 먼곳을 찾는다
      there = (next + arr[next] > next_pot) ? next : there;
      next_pot = (next + arr[next] > next_pot) ? next + arr[next] : next_pot;
    }
    if (next_pot == here) return -1;        // 이동이 없었다면 return -1
    ++jumps;
    here = there;
    next_pot = here+1;
  }
  return jumps;
}

int main(void) { 
   
  // keep this function call here
  int A[] = coderbyteInternalStdinFunction(stdin);
  int arrLength = sizeof(A) / sizeof(*A);
  cout << ArrayMinJumps(A, arrLength);
  return 0;
    
}
```