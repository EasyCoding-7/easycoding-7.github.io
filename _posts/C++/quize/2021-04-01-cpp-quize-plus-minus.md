---
layout: post
title:  "(C++ : Quize) Plus Minus"
summary: ""
author: C++
date: '2021-04-01 0:00:00 +0000'
category: ['Cpp', 'Quize']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Quize/plus-minus/
---

## Q

숫자사이에 +, -를 넣어서 0이될수 있는지 판별<br>
만약 두 개 이상의 가능한 조건이 있다면 마이너스가 많은 쪽이 정답<br>

```
Input: 199
Output: not possible
```

```
Input: 26712
Output: -+--
```

---

## Write Code

* [Test Link](https://ideone.com/)

```cpp
#include <iostream>
#include <string>
using namespace std;

string PlusMinus(int num) {
  return ""; 
}

int main(void) { 
  cout << PlusMinus(199);
  return 0;
}
```

---

## A

재귀적으로 해결해야한다.<br>
필요한 변수는<br>

* 총 합 : 0이 되었는지 확인용
* 원본 숫자 : 재귀를 돌리려면 필요
* 현재 마이너스 개수 : 최대 마이너스 개수 확인용
* 최대 마이너스 개수 : 현재 마이너스 개수와 비교용
* 리턴 : 리턴용
* 리턴 템프 : 최대 마이너스 갱신시 리턴 템프를 리턴으로 변환

---

## A - Code

```cpp
#include <iostream>
#include <string>
using namespace std;

void helper(int sum, string &source, string &res, string temp, int minus, int &max) {
  if(temp.size()==source.size()-1) {        // 모든 기호가 다 들어감.
    if(sum == 0 &&          // 모든 합이 0이고
        minus > max) {      // 최대 마이너스 개수를 체크(두 번째 조건임)
      max = minus; 
      res = temp;
    }
    return;
  }

  // 최대 -가 우선되기에 -를 먼저 검사
  temp.push_back('-');         
  int sum1 = sum - (source[temp.size()] - '0');
  helper(sum1, source, res, temp, minus+1, max);
  temp.pop_back();

  temp.push_back('+');
  int sum2 = sum + (source[temp.size()] - '0');
  helper(sum2, source, res, temp, minus, max);
  return; 
}

/*
145라 가정 분기가 된다 생각하면 좀 쉽다

''  -> '-'  -> '--'  --> 이걸 검사 후 조건에 맞으면 max, res에 값을 갱신한다
            -> '-+'
    -> '+'  -> '+-'
            -> '++'
*/

string PlusMinus(int num) {
  string no = "not possible";
  string source = to_string(num);
  if(source.size() == 1) return no; 
  if(source.empty()) return no;
  string res = "";
  string temp = "";
  int sum = source[0] - '0';
  int max = 0; 
  helper(sum, source, res, temp, 0, max);
  if (res == "") return no;
  else return res; 
}

int main(void) { 
   
  // keep this function call here
  cout << PlusMinus(coderbyteInternalStdinFunction(stdin));
  return 0;
    
}
```