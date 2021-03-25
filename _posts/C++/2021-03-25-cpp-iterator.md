---
layout: post
title:  "(C++) Iterator"
summary: ""
author: C++
date: '2021-03-25 0:00:00 +0530'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
keywords: ['iterator']
usemathjax: false
permalink: /blog/cpp/iterator/
---

## iterator는 왜 쓸까?

* list와 vector에 있는 요소를 열거하는 방법은 다르다.(메모리구조로 인해)
* 동일한 방법으로 요소에 접근할 수 있을까? -> iterator 적용

```cpp
#include <vector>
#include <iostream>

using namespace std;

int main() {
	vector<int> vec = {1, 2, 3};
	for (auto it = vec.begin(); it != vec.end(); ++it) {
		cout << *it << endl;
	}
	
	// C++11 range-for loop
	for (auto v: vec) {
		cout << v << endl;
	}
}
```

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main() {
	vector<int> vec = {1, 2, 3};

	// Create an iterator which points at the first element in vec
	vector<int>::const_iterator it = vec.cbegin();
	
    // Create a const iterator which points at the last element in vec
	vector<int>::const_reverse_iterator it = vec.rbegin();

    // Create an iterator which points at the last element in vec
	vector<int>::reverse_iterator it = vec.rbegin();

	// C++11 alternative
	//auto it = vec.cbegin();
	
	while (it != vec.cend()) {          // Check whether the iterator is still valid
	    cout << *it << endl;            // Print out the data in the first element - displays 1
	    //*it = 6;                      // Compiler error! Cannot modify through const iterator
	    ++it;                           // Advance the iterator to the next element
	}
}
```

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main() {
	vector<int> vec = {1, 2, 3};                  // Create a vector

	auto ins_it = vec.begin();                    // Get an iterator to the first element
	++ins_it;                                     // Advance it to the second element
	
	auto it = inserter(vec, ins_it);              // Get an insert iterator for vec

	// Assign to this iterator
	*it = 99;                                     // Adds element with value 99 before second element
	*it = 88;                                     // Adds element 88 to vec, before the same element
	
	// vec  now contains {1, 99, 88, 2, 3}

	// Print out vector elements
	for (auto v: vec)
		cout << v << endl;
}
```

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main() {
	vector<int> vec;                        // Create an empty vector

	auto it = back_inserter(vec);           // Get an insert iterator for vec

	// Assign to this iterator
	*it = 99;                               // Adds element with value 99 to vec
	*it = 88;                               // Adds element 88 to vec, which now contains {99, 88}

	// Print out vector elements
	for (auto v: vec)
		cout << v << endl;
}
```

---

## 여기서 부터는 iterator구현기 ... 필요하다면 보자

간략하게 iterator를 구현해보자면 아래와 같다(참고만 하자)

```cpp
#include <iostream>
using namespace std;

template<typename T> struct Node
{
    T data;
    Node* next;
    Node(const T& d, Node* n) : data(d), next(n) {}
};

template<typename T> class slist
{
    Node<T>* head = 0;
public:
    void push_front(const T& n) { head = new Node<T>(n, head); }
    T front()                   { return head->data; }
};

int main()
{
    slist<int> s;

    s.push_front(10);
    s.push_front(30);
    s.push_front(40);
    s.push_front(50);
}
```

구현시작

---

```cpp
// 반복자의 규칙
template<typename T> 
struct IEnumerator
{
    virtual ~IEnumerator() {}
    virtual bool MoveNext() = 0;
    virtual T& GetObject() = 0;
};

// slist의 반복자
template<typename T> class SlistEnumerator : public IEnumerator<T>
{
    Note<T>* current = 0;
public: 
    SlistEnumerator( Node<T>* p = 0 ) : current(p) {}

    virtual bool MoveNext() { 
        current = current->next; 
        return current; 
    }
    virtual T& GetObject() { return current->data; }
};

// 모든 컨테이너에서 반복자를 꺼낼 수 있어야 한다.
// 컨테이너가 지켜야하는 인터페이스
template<typename T> struct IEnumerable
{
    virtual ~IEnumerable() {}
    virtual IEnumerator<T>* GetEnumerator() = 0;
};

template<typename T> class slist : public IEnumerable<T>
{
    Node<T>* head = 0;
public:
    virtual IEnumerator<T>* GetEnumerator()
    {
        return new SlistEnumerator<T>(head);
    }
    void push_front(const T& n) { head = new Node<T>(n, head); }
    T front()                   { return head->data; }
};

int main()
{
    slist<int> s;

    s.push_front(10);
    s.push_front(20);

    IEnumerator<int>* p = s.GetDnumerator();

    int n = p->GetObject();
    p->MoveNext();
}
```

## STL 방식의 iteratorPermalink

* 인터페이스를 사용하지 않는다.
* 이동 및 접근 함수는 포인터에 규칙에 따른다. (++, –, * 사용가능하게 만든다)

```cpp
template<typename T> class slist_iterator
{
    Note<T>* current = 0;
public: 
    slist_iterator( Node<T>* p = 0 ) : current(p) {}

    inline slist_iterator& operator++()
    { 
        current = current->next; 
        return *this*; 
    }
    inline T& operator*() { return current->data; }
};
```

* 컨테이너의 경우 지켜야하는 규칙을 담을 인터페이스를 없앤다.
* 인터페이스는 없지만 약속된 함수인 begin멤버 함수로 반복자를 꺼낸다.

```cpp
template<typename T> class slist
{
    Node<T>* head = 0;
public:
    slist_iterator<T> begin()
    {
        return slist_iterator<T>(head);
    }
    void push_front(const T& n) { head = new Node<T>(n, head); }
    T front()                   { return head->data; }
};
```

```cpp
int main()
{
    slist<int> s;

    s.push_front(10);
    s.push_front(20);

    slist_iterator<int> p = s.begin();

    cout << *p << endl;
    ++p;
}
```