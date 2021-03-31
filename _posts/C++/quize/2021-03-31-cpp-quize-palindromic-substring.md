---
layout: post
title:  "(C++ : Quize) Palindromic Substring"
summary: ""
author: C++
date: '2021-03-31 0:00:00 +0000'
category: ['Cpp', 'Quize']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Quize/palindromic-substring/
---

## Q

문자열 가운데 대칭된 문자열을 찾아라, 없다면 none

```
Input: "hellosannasmith"
Output: sannas
```

```
Input: "abcdefgg"
Output: none
```

---

## Write Code

* [Test Link](https://ideone.com/)

```cpp
#include <iostream>
#include <string>
using namespace std;

string PalindromicSubstring(string str) {
  // code
  return str;
}

int main(void) { 
   
  // keep this function call here
  cout << PalindromicSubstring("hellosannasmith");
  return 0;
}
```

---

## A

for문을 두 개 돌리며 하나씩 비교해가는것이 핵심.

```cpp
  for (int length = str.length(); length > 1 ; --length) {
    for (int i = 0; i < str.length()-length; ++i) {
```

---

## A - Code

```cpp
#include <iostream>
#include <string>
#include <algorithm>
using namespace std;

bool isPalindrome(std::string word) {
  std::string word2 = word;
  std::reverse(word2.begin(), word2.end());
  return word == word2;
}

string PalindromicSubstring(string str) {
  for (int length = str.length(); length > 1 ; --length) {
    for (int i = 0; i < str.length()-length; ++i) {
      std::string word = str.substr(i, length);
      if (isPalindrome(word)) {
        return word;
      }
    }
  }  
  return "none";

}

int main(void) { 
   
  // keep this function call here
  cout << PalindromicSubstring(coderbyteInternalStdinFunction(stdin));
  return 0;
}
```