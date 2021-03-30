---
layout: post
title:  "(C++ : Quize) Overlapping Rectangles"
summary: ""
author: C++
date: '2021-03-29 0:00:00 +0000'
category: ['Cpp', 'Quize']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Quize/most-free-time/
---

## Q

```
Input: {"(0,0),(0,-2),(3,0),(3,-2),(2,-1),(3,-1),(2,3),(3,3)"}
Output: 6
```

```
Input: {"(0,0),(5,0),(0,2),(5,2),(2,1),(5,1),(2,-1),(5,-1)"}
Output: 3
```

* [Test Link](https://ideone.com/)

```cpp
#include <iostream>
#include <string>
using namespace std;

string OverlappingRectangles(string strArr[], int arrLength) {
  
  // code goes here  
  return strArr[0];

}

int main(void) { 
   
  // keep this function call here
  string A[] = {"(0,0),(0,-2),(3,0),(3,-2),(2,-1),(3,-1),(2,3),(3,3)"};
  int arrLength = sizeof(A) / sizeof(*A);
  cout << OverlappingRectangles(A, arrLength);
  return 0;
    
}
```

---

## A

시작은 문자열을 파싱하는 것 부터 x, y를 벡터로 뽑아내 보자.

```cpp
void Split(string str, vector<int> &x, vector<int> &y)
{

  for(size_t i=0;i<str.length();i++)
  {
    // 괄호를 제거
    if(str[i]=='('||str[i]==')')
      str.erase(i,1);
  }
  vector<int> v;
  size_t first = 0;
  while(first<str.length())
  {
    size_t last = first;
    // 숫자가 한 자리만 들어온다는 보장이 없기에 아래와 같이 처리
    while(last<str.length()&&str[last]!=',')
      last++;
    if(last>first)
    {
      int value = stoi(str.substr(first,last-first));
      // 순서대로 x, y가 들어간다.
      v.push_back(value);
    }
    
    if(last<str.length())
      last++;
    first = last;
  }

  for(size_t i=0;i<v.size();i++)
  {
      // 다시 x, y를 각각 벡터에 넣는다
    if(i%2==0)
      x.push_back(v[i]);
    else
      y.push_back(v[i]);
  }

  return;
}
```

네모의 left, right, top, bottom을 알아야 겹치는 부분을 구할수 있다.

```cpp
  for(size_t i=0;i<4;i++)
  {
    rect1.left = rect1.left<x[i]?rect1.left:x[i];       // 가장 작은 x가 left
    rect1.top = rect1.top<y[i]?rect1.top:y[i];          // 가장 작은 y가 top
    rect1.right = rect1.right>x[i]?rect1.right:x[i];    // 가장 큰 x가 right
    rect1.bottom = rect1.bottom>y[i]?rect1.bottom:y[i]; // 가장 큰 y가 bottom

    rect2.left = rect2.left<x[i+4]?rect2.left:x[i+4];
    rect2.top = rect2.top<y[i+4]?rect2.top:y[i+4];
    rect2.right = rect2.right>x[i+4]?rect2.right:x[i+4];
    rect2.bottom = rect2.bottom>y[i+4]?rect2.bottom:y[i+4];
  }
```

겹치는 네모를 구한다.

```cpp
rectangle FindOverlap(rectangle r1,rectangle r2)
{
  rectangle overlap;
  overlap.left = r1.left>r2.left?r1.left:r2.left;            // 두 네모 중 더 큰 부분이 left
  overlap.top = r1.top>r2.top?r1.top:r2.top;                 // 두 네모 중 더 큰 부분이 top
  overlap.right = r1.right<r2.right?r1.right:r2.right;       // 두 네모 중 더 작은 부분이 right
  overlap.bottom = r1.bottom<r2.bottom?r1.bottom:r2.bottom;  // 두 네모 중 더 작은 부분이 bottom

  if(overlap.left>overlap.right||overlap.top>overlap.bottom)
  {
    initialRectangle(overlap);
  }
  return overlap;
}
```

이제 오버랩 된 네모의 크기를 구하면 끝

---

## A - Code

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

struct rectangle
{
  int left;
  int top;
  int right;
  int bottom;
};

void ShowRectangle(rectangle r)
{
  cout <<"("<< r.left<<"," << r.top<<"," << r.right<<"," << r.bottom<<")" << endl;
}

void initialRectangle(rectangle &r)
{
  r.left = 1e9;
  r.top = 1e9;
  r.right = -1e9;
  r.bottom = -1e9;
}

void Split(string str, vector<int> &x, vector<int> &y)
{

  for(size_t i=0;i<str.length();i++)
  {
    if(str[i]=='('||str[i]==')')
      str.erase(i,1);
  }
  vector<int> v;
  size_t first = 0;
  while(first<str.length())
  {
    size_t last = first;
    while(last<str.length()&&str[last]!=',')
      last++;
    if(last>first)
    {
      int value = stoi(str.substr(first,last-first));
      v.push_back(value);
    }
    
    if(last<str.length())
      last++;
    first = last;
  }

  for(size_t i=0;i<v.size();i++)
  {
    if(i%2==0)
      x.push_back(v[i]);
    else
      y.push_back(v[i]);
  }

  return;
}

rectangle FindOverlap(rectangle r1,rectangle r2)
{
  rectangle overlap;
  overlap.left = r1.left>r2.left?r1.left:r2.left;
  overlap.top = r1.top>r2.top?r1.top:r2.top;
  overlap.right = r1.right<r2.right?r1.right:r2.right;
  overlap.bottom = r1.bottom<r2.bottom?r1.bottom:r2.bottom;

  if(overlap.left>overlap.right||overlap.top>overlap.bottom)
  {
    initialRectangle(overlap);
  }
  return overlap;
}

int AreaRectangle(rectangle r)
{
  int width = r.right - r.left;
  int height = r.bottom - r.top;

  return width*height;
}

string OverlappingRectangles(string strArr[], int arrLength) {
  
  // code goes here  
  rectangle rect1,rect2;
  initialRectangle(rect1);
  initialRectangle(rect2);

  vector<int> x, y;
  Split(strArr[0],x,y);
  for(size_t i=0;i<4;i++)
  {
    rect1.left = rect1.left<x[i]?rect1.left:x[i];
    rect1.top = rect1.top<y[i]?rect1.top:y[i];
    rect1.right = rect1.right>x[i]?rect1.right:x[i];
    rect1.bottom = rect1.bottom>y[i]?rect1.bottom:y[i];

    rect2.left = rect2.left<x[i+4]?rect2.left:x[i+4];
    rect2.top = rect2.top<y[i+4]?rect2.top:y[i+4];
    rect2.right = rect2.right>x[i+4]?rect2.right:x[i+4];
    rect2.bottom = rect2.bottom>y[i+4]?rect2.bottom:y[i+4];
  }
  rectangle overlap = FindOverlap(rect1,rect2);
  int areaOverlap = AreaRectangle(overlap);
  int areaRect1 = AreaRectangle(rect1);

  if(areaOverlap<=0)
    return "0";
  // ShowRectangle(rect1);
  // ShowRectangle(rect2);
  // ShowRectangle(overlap);

  return to_string(areaRect1/areaOverlap);

}

int main(void) { 
   
  // keep this function call here
  string A[] = coderbyteInternalStdinFunction(stdin);
  int arrLength = sizeof(A) / sizeof(*A);
  cout << OverlappingRectangles(A, arrLength);
  return 0;
    
}
```