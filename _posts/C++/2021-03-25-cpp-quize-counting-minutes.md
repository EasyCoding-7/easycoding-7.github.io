---
layout: post
title:  "(C++ : Quize) Counting Minutes"
summary: ""
author: C++
date: '2021-03-25 0:00:00 +0530'
category: ['Cpp', 'Quize']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Quize/counting-minutes/
---

## Q

```
Input: "12:30pm-12:00am"
Output: 690
```


```
Input: "1:23am-1:08am"
Output: 1425
```

---

## A

```cpp
#include <iostream>
#include <string>
using namespace std;

int FormateTime(string str)
{
  int hour,minute;
  size_t pos = str.find(":");
  hour = stoi(str.substr(0,pos));
  minute = stoi(str.substr(pos+1,2));

  string period = str.substr(str.length()-2,2);
  if(period=="pm"&&hour<12)
  {
    hour += 12;
  }
  else if(period=="am"&&hour==12)
  {
    hour = 24;
  }
  return hour*60+minute;
}

string CountingMinutes(string str) {
  
  // code goes here  
  size_t pos = str.find("-");
  string start = str.substr(0,pos);
  string end = str.substr(pos+1,str.length()-pos-1);
  
  int timeStart = FormateTime(start);
  int timeEnd = FormateTime(end);
  
  if(timeEnd<=timeStart)
  {
    timeEnd += 24*60;
  }
  return to_string(timeEnd-timeStart);

}

int main(void) { 
   
  // keep this function call here
  cout << CountingMinutes(coderbyteInternalStdinFunction(stdin));
  return 0;
    
}
```