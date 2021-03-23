---
layout: post
title:  "(C++ : Quize) Matching Characters"
summary: ""
author: C++
date: '2021-03-23 0:00:00 +0530'
category: ['Cpp', 'Quize']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Quize/matching-characters/
---

## Q

공통된 문자 사이의 중복되지 않은 최대 문자를 출력

```
Input: "mmmerme"
Output: 3
```

```
Input: "abccdefghi"
Output: 0
```

---

## A

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

vector<string> Split(string str)
{
  vector<string> v;
  for(size_t i=0;i<str.length();i++)
  {
    for(size_t j=i+1;j<str.length();j++)
    {
      if(str[j]==str[i]&&j-i>1)
      {
        v.push_back(str.substr(i+1,j-i-1));
      }
    }
  }
  return v;
}

int CheckUnique(string str)
{
  int length = str.length();

  for(size_t i=0;i<str.length();i++)
  {
    for(size_t j=i+1;j<str.length();j++)
    {
      if(str[j]==str[i]) 
      {
        length--;
      }
    }
  }
 
  return length;
}

string MatchingCharacters(string str) {
  
  // code goes here  
  vector<string> unique = Split(str);
  if(unique.size()==0) return "0";
  int max = 0;
  for(size_t i=0;i<unique.size();i++)
  {
    if(CheckUnique(unique[i])>max)
    {
      max = CheckUnique(unique[i]);
    }
  }


  // free(count);
  return to_string(max);

}

int main(void) { 
   
  // keep this function call here
  cout << MatchingCharacters(coderbyteInternalStdinFunction(stdin));
  return 0;
    
}
```