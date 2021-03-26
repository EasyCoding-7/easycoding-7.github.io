---
layout: post
title:  "(C++) String"
summary: ""
author: C++
date: '2021-03-25 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
keywords: ['string']
usemathjax: false
permalink: /blog/cpp/string/
---

## substr

```cpp
#include <iostream>
#include <string>
 
using namespace std;
 
int main() {
	string str {"Hello world"};
    str[1] = 'a';
    string s1 = str.substr(6);
    string s2 = str.substr(6,2);
	cout << str << endl;        // Hallo world
	cout << s1 << endl;         // world
	cout << s2 << endl;         // wo
 
	return 0;
}
```

---

## find, rfind

```cpp
void find() {
	string str {"Hello world"};
	size_t pos = str.find('o');
	
	if (pos != string::npos) {
		str[pos] = 'p';
	}
	else {
		cout << "Could not find the search string\n";
	}
	cout << str << endl;
}

void rfind() {
	string str {"Hello world"};
	size_t pos = str.rfind('o');

	if (pos != string::npos) {
		str[pos] = 'p';
	}
	else {
		cout << "Could not find the search string\n";
	}
	cout << str << endl;
}
```

---

## find option

```cpp
#include <iostream>
#include <string>

using namespace std;

int main() {
	string str {"Hello world"};
    /*
    H e l l o   w o r l d
    0 1 2 3 4 5 6 7 8 9 10
    */
	string vowels {"aeiou"};
	cout << str.find_first_of(vowels) << endl;      // 1
	cout << str.find_last_of(vowels) << endl;       // 7
	cout << str.find_first_not_of(vowels) << endl;  // 0
	cout << str.find_last_not_of(vowels) << endl;   // 10

	return 0;
}
```

---

## append

```cpp
#include <iostream>
using namespace std;
 
int main() {
	string hello {"Hello"};
	hello.append(" world");
	cout << "hello = " << hello << endl;		// hello = Hello world
	string hello2 {"Hello"};
	hello2.append("wow!!!!", 3, 2);				// hello2 = Hello!!
	cout << "hello2 = " << hello2 << endl;

	return 0;
}
```

---

## insert

```cpp
#include <iostream>
using namespace std;

int main() {
	string hello { "hello" };
	hello.insert(4, "eh");
	hello.insert(6, 3, '.');
	hello.insert(6, 1, '.');
	cout << "hello = " << hello << endl;		// hello = helleh....o
	return 0;
}
```

```cpp
#include <iostream>
using namespace std;

int main() {
	string hello { "hello" };
	hello.insert(4, "eh");			// helleho
	cout << "1 : " << hello << endl;
	hello.insert(6, 3, '.');		// helleh...o
	cout << "2 : " << hello << endl;
	hello.insert(6, 1, '.');		// helleh....o
	cout << "3 : " << hello << endl;
	
	string dol {"Dolly"};		// oll
	hello.insert(4, dol, 1, 3);		// hellolleh....o
	cout << "4 : " << hello << endl;
	size_t opos = hello.find('o');                                       
	if (opos != string::npos)
		hello.insert(opos, "OOO");
	cout << "hello = " << hello << endl;

	return 0;
}
```

---

## char[] 보다 장점이 있나?

```cpp
#include <iostream>

int main()
{
    char s1[10] = "hello";
    char s2[10];

    s2 = s1;        // error - strcpy(s2, s1)

    if (s1 == s2)   // error strcmp(s1, s2)
    {

    }
}
```

저런 간단한 비교조차 못함 -> string을 쓰자

---

## string to const char*

```cpp
#include <iostream>
#include <string.h>
using namespace std;

int main()
{
    string s1 = "hello";
    char s2[10];

    strcpy(s2, s1);             // error - string to const char* 변환필요
    strcpy(s2, s1.c_str());     // ok
```

---

## string to double

```cpp
#include <iostream>
#include <string.h>
using namespace std;

int main()
{
    // string to double
    string s3 = "3.4";
    double d = stod(s3);

    // double to string
    string s4 = to_string(5.4);
```

---

## 접미사를 통한 자료형 선택

```cpp
#include <iostream>
#include <string.h>
using namespace std;
using namespace std::string_literals;

void foo(string s) { cout << "string" << endl; }
void foo(const char * s) { cout << "char" << endl; }

int main()
{
    foo("hello");   // char
    foo("hello"s);  // string -> 접미사로 자료형 선택이 가능
}
```

* “string”s - string
* 0s - seconds
* 0i - complex
