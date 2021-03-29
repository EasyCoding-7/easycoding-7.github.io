---
layout: post
title:  "(C++ : Quize) Most Free Time"
summary: ""
author: C++
date: '2021-03-26 0:00:00 +0000'
category: ['Cpp', 'Quize']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Quize/most-free-time/
---

## Q

```
Input: {"12:15PM-02:00PM","09:00AM-10:00AM","10:30AM-12:00PM"}
Output: 00:30
```

```
Input: {"12:15PM-02:00PM","09:00AM-12:11PM","02:02PM-04:00PM"}
Output: 00:04
```

* [Test Link](https://ideone.com/)

```cpp
#include <iostream>
using namespace std;

string MostFreeTime(string strArr[], int n) {
  return "";
}

int main() {

  string A[] = {"12:15PM-02:00PM","09:00AM-10:00AM","10:30AM-12:00PM"};
  int arrLength = sizeof(A) / sizeof(*A);
  cout << MostFreeTime(A, arrLength);
  return 0;
}
```

---

## A

일단 시간을 파싱해야 한다. 우선 나에게 필요한것은 시간이 아니라 분이기에<br>
예를 들어 12:15PM-02:00PM을 파싱한다면,<br>

```cpp
int timeHelper(string time) {
  int temp = 0;
  int hour = stoi(time.substr(0, 2));
  if (hour < 12) {
    temp += hour * 60;
  }
  temp += stoi(time.substr(3, 2));
  if (time[5] == 'P') {
    temp += 12 * 60;
  }
  return temp;
}
```

여기까지 진행하면 시작시간과 종료시간을 분으로 변환이 가능하다<br>
시작시간과 종료시간을 벡터에 담고<br>

```cpp
  vector<pair<int, int>> minutes;
  for (int i = 0; i != n; ++i) {
    int time_begin = timeHelper(strArr[i].substr(0, 7));
    int time_end = timeHelper(strArr[i].substr(8, 7));
    minutes.push_back({time_begin, time_end});
  }
```

시작시간으로 정렬을 한다.<br>

```cpp
sort(minutes.begin(), minutes.end());
```

시작시간으로 정렬을 해도 되는 이유가 종료시간과 다음 시작시간이 겹치는 경우는 없다(몸이 두 개이지 않은 이상)<br>
이후 다음 시작 시간과 현재 종료시간이 가장 긴 것을 찾는다.

```cpp
  sort(minutes.begin(), minutes.end());
  int most = 0;
  for (int i = 0; i != n - 1; ++i) {
    most = max(most, minutes[i + 1].first - minutes[i].second);
  }
```

마지막으로 다시 분을 시간으로 계산하며 끝낸다.

```cpp
  string res = "";
  int hour = most / 60, minute = most % 60;
  if (hour < 10) {
    res += '0';
  }
  res += to_string(hour) + ':';
  if (minute < 10) {
    res += '0';
  }
  res += to_string(minute);
  
  return res;
```

---

## A - Code

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int timeHelper(string time) {
  int temp = 0;
  int hour = stoi(time.substr(0, 2));
  if (hour < 12) {
    temp += hour * 60;
  }
  temp += stoi(time.substr(3, 2));
  if (time[5] == 'P') {
    temp += 12 * 60;
  }
  return temp;
}

string MostFreeTime(string strArr[], int n) {
  
  // code goes here  
  vector<pair<int, int>> minutes;
  for (int i = 0; i != n; ++i) {
    int time_begin = timeHelper(strArr[i].substr(0, 7));
    int time_end = timeHelper(strArr[i].substr(8, 7));
    minutes.push_back({time_begin, time_end});
  }

  sort(minutes.begin(), minutes.end());
  int most = 0;
  for (int i = 0; i != n - 1; ++i) {
    most = max(most, minutes[i + 1].first - minutes[i].second);
  }

  string res = "";
  int hour = most / 60, minute = most % 60;
  if (hour < 10) {
    res += '0';
  }
  res += to_string(hour) + ':';
  if (minute < 10) {
    res += '0';
  }
  res += to_string(minute);
  
  return res;
}

int main(void) { 
   
  // keep this function call here
  /* Note: In C++ you first have to initialize an array and set 
     it equal to the stdin to test your code with arrays. */

  string A[] = gets(stdin);
  int arrLength = sizeof(A) / sizeof(*A);
  cout << MostFreeTime(A, arrLength);
  return 0;
    
}
```