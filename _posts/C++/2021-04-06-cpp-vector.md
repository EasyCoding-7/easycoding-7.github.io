---
layout: post
title:  "(C++) Vector"
summary: ""
author: C++
date: '2021-04-06 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
keywords: ['vector']
usemathjax: false
permalink: /blog/cpp/vector/
---

## vector 이론적 부분

* vector의 동작 원리를 이해 한다.(size/capacity)
* 중간 삽입/삭제를 이해한다.
* 처음/끝 삽입/삭제를 이해한다.
* 임의 접근을 이해한다.

### vector의 동작 원리

1. 여유분을 두고 메모리를 할당한다.
2. 여유분까지 꽉 찼으면, 메모리를 증설한다.

여기서 증설이라 함은 기존의 영역을 버리고 새로운 메모리를 할당함을 의미한다.

그렇다면 의문점은

1. 여유분은 얼마나 새로 할당해주는데?
2. 증설은 얼마나 더 해주는데?
3. 기존의 메모리는 어떻게 처리하는데?

참고,

* `size()` : 실제 데이터가 들어가 있는 메모리 수
* `capacity()` : 여유분을 포함한 메모리 수

```cpp
// 이걸 한 번 실행해 보자

vector<int> v;

for(int i = 0; i < 1000; i++)
{
    v.push_back(100);
    cout << v.size() << " " << v.capacity() << endl;
}
```

처음에는 size, capacity가 동일하게 증가하다. 어느 순간부터 capacity가 훨씬 크게 증가한다.<br>
대략적으로 capacity * 1.5만큼 capacity를 잡게되는데 이는 컴파일러에 따라 다르다.<br>

1, 2번은 해결됐고, 기존메모리는 버려지고 새로운 메모리로 잡게 되는데... 새로운 메모리를 자주 잡을 수 록 낭비되는 자원도 함께 늘어난다. -> 처음에 메모리를 잘 잡자

```cpp
// iterator에 대해 잠깐보자

vector<int> v(10);

// vector<int>::size_type로 카운터 변수를 잡는다.
for(vector<int>::size_type i = 0; i < v.size(); i++)
{
    // ...
}

vector<int>::iterator it;
it = v.begin(); // iterator를 받는다
cout << (*it) << endl;

// const iterator로 받는 방법도 있다
vector<int>::const_iterator cit1 = b.cbegin();
*cit1 = 10; // error
```

---

## vector 사용적 부분

## iterator 활용

```cpp
#include <iostream>
#include <vector>

using namespace std;

class MyClass
{
public:
	int a = 0;
	int b = 0;
};

int main()
{
	vector<MyClass*> vec;

	MyClass* mc = new MyClass;
	vec.push_back(mc);

	MyClass* mc2 = new MyClass;
	vec.push_back(mc2);

	auto it = vec.begin();

	int aa = (*it)->a;
	int bb = (*it)->b;

    // 삭제
    int index = 1;
    vec.erase(vec.begin() + index);
}
```

---

## Foreach활용

```cpp
// 과거 스타일 for문
for (int i = 0; i < 10; i++)
{
    std::cout << i << std::endl;
}

// C++11 스타일 for문
for(int i : {0, 1, 2, 3, 4, 5})
{
    std::cout << i << std::endl;
}

std::vector<std::string> name_vector;       // 아마 보통이 방법을 많이 쓸 듯!
for(const auto& element : name_vector)
{
    std::cout << element << std::endl;
}

// std::for_each 활용
std::vector<std::string> name_vector{ "test1", "test2", "test3" };
std::for_each(name_vector.begin(), name_vector.end(), [](auto& input) {std::cout << input << std::endl; });

void print(std::string& input)
{
    std::cout << input << std::endl;
}
std::vector<std::string> name_vector{ "test1", "test2", "test3" };
std::for_each(name_vector.begin(), name_vector.end(), print);
```

---

## 초기화

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main()
{
    vector<int> v1;
    vector<int> v2(10);
    vector<int> v3(10, 3);
    vector<int> v4(v3);

    vector<int> v5 = {1,2,3,4};
    vector<int> v6{1,2,3,4};

    vector<int> v7(10,0);       // 10개를 0으로 초기화
    vector<int> v8{10,0};       // 2개를 10, 0으로 초기화

    /* *************************** */

    // v.push_front(10);       // error
    v.push_back(10);
    v.insert(begin(v)+1, 30);   // insert(위치, 값)

    /* *************************** */

    int n = v.front();
    int n1 = v[0];

    /* *************************** */

    int x[5] = {1,2,3,4,5};
    v.assing(x, x+5);

    /* *************************** */

    v[100] = 10;        // 예외 없이 runtime error
    v.at(100) = 10;     // error!

    for(int i = 0; i < v.size(); i++)
        v[i] = 10;      // ok
        v.at(i) = 10;   // 성능이 나쁘다.
}
```

---

## 멤버삽입

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main()
{
    vector<int> v(10, 0);
    v.resize(7);        // 메모리를 재할당할 것인가? or 메모리 사용량을 줄일 것인가?
    // 메모리 사용량을 줄이게 된다.

    cout << v.size() << endl;           // 7
    cout << v.capacity() << endl;       // 10

    v.push_back();

    cout << v.size() << endl;           // 8
    cout << v.capacity() << endl;       // 10

    v.pop_back()

    cout << v.size() << endl;           // 7
    cout << v.capacity() << endl;       // 10

    v.shrink_to_fit();
    cout << v.size() << endl;           // 7
    cout << v.capacity() << endl;       // 7

    v.push_back(10);
    cout << v.size() << endl;           // 8
    cout << v.capacity() << endl;       // 10 ~ 11 : 1.5배로 할당하게 된다.
}
```

---

## 초기화2

vector와 c스타일 호환

```cpp
#include <iostream>
#include <vector>
using namespace std;

void print(int* arr, int sz)
{
    for(int i = 0; i < sz; i++)
        cout << arr[i] << endl;
}

int main()
{
    int arr[10] = {1,2,3,4,5,6,7,8,9,10};
    print(arr, 10);

    vector<int> v = {1,2,3,4,5,6,7,8,9,10};
    // print(v,v.size());        // error!
    print(v.data(), v.size());   // 메모리 주소값을 넘기게 된다.
}
```

---

## Example

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include <fstream>
using namespace std;

struct FindChar
{
    string data;
    FindChar(string s) : data(s) {}

    bool operator()(char c) const
    {
        auto p = find(begin(data), end(data), c);
        return p != end(data);
    }
};

// FindChar fc("0123456789");

int main()
{
    ifstream f("vector4.cpp");
    string s;
    f >> s;         // 단어를 받는다.
    getline(f, s);  // 한 문장을 받는다.
    while(getline(f,s))
        cout << s << endl;      // 이런식으로 많이 받음.

    cout << s << endl;      // #include 출력

    while(getline(f,s))
        v.push_back(s);

    reverse(begin(v), end(v));      // 파일의 내용을 뒤집는다.
    for(int i = 0; i < v.size(); i++)
        reverse(begin(v[0], end(v[0])));// 각 줄을 뒤집는다.
        replace(begin(v[i]], end(v[i]), 'i', ' ')); // i를 모두 공백을 변경한다.
        replace_if(begin(v[i]], end(v[i]), [](char c){return c == 'a';}], ' '));
        FindChar fc("aieouAIEOU");
        replace_if(begin(v[i]), end(v[i]), fc, ' ');

    for(auto str : v)
        cout << str << endl;    // vector의 모든 내용 출력
}
```

```cpp
// 아래와 같은 모양으로 헤쉬테이블을 만들수 있음.
vector<list<int>> h(10);
h[0].push_back(10);
h[0].push_back(20);
```

---

