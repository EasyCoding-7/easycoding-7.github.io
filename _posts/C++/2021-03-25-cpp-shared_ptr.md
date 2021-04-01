---
layout: post
title:  "(C++) shared_ptr"
summary: ""
author: C++
date: '2021-03-25 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
keywords: ['shared_ptr']
usemathjax: true
permalink: /blog/cpp/shared_ptr/
---

## 이렇게 쓰면 좋겠지?

```cpp
#include <iostream>
#include <memory>
using namespace std;

class cl1
{
public:
	cl1() { cout << "cl1()" << endl; }
	~cl1() { cout << "~cl1()" << endl; } 
    void printCL1() { cout << "Hello This is CL!" << endl; }
};

class parent1
{
public:
	parent1(int num) { cout << "parent1() : " << num << endl; m_num = num; }
	~parent1() { cout << "~parent1() : " << m_num  << endl; }
	void SetCL1(shared_ptr<cl1> _cl1) { m_cl1 = _cl1; }
    void PrintCL1() { m_cl1->printCL1(); }

private:
	shared_ptr<cl1> m_cl1;
    int m_num = -1;
};

int main() {
	// your code goes here
	shared_ptr<cl1> m_cl1 = make_shared<cl1>();

	unique_ptr<parent1> m_pr1 = make_unique<parent1>(1);
	parent1* m_pr2 = new parent1(2);

	m_pr1->SetCL1(m_cl1);
	m_pr2->SetCL1(m_cl1);

    m_pr1->PrintCL1();
    m_pr2->PrintCL1();

	return 0;
}
```

* 해당 클래스에서 사용하는 포인터 : `unique_ptr`
* 다른 클래스에서 참조해야할 포인터 : `shared_ptr`

```
cl1()
parent1() : 1
parent1() : 2
Hello This is CL!
Hello This is CL!
~parent1() : 1
```

---

```cpp
#include <iostream>
#include <memory>

using namespace std;

int main() {
	auto ptr{make_shared<int>(36)};
	cout << "shared_ptr's data is " << *ptr << endl;
	
	auto ptr2 = ptr;
	cout << "Copied shared_ptr's data is " << *ptr << endl;
	
	shared_ptr<int> ptr3;
	ptr3 = ptr;
	cout << "Assigned shared_ptr's data is " << *ptr << endl;
}
```

## 언제쓰냐에 제일 좋은 답변

어쨋든 스마트 포인터가 사용자가 delete를 안하게 하고 싶다는게 핵심이다.

```cpp
#include <iostream>
#include <memory>
#include <vector>
using namespace std;

class Car
{
    int color;
    int speed;
public:
    ~Car() { cout << "~Car()" << endl; }
 
    void Go() { cout << "Car go" << endl; }
};

void foo(Car* p)
{
    cout << "Delete Car" << endl;
}

int main() {
	vector<shared_ptr<Car>> m_carVec;

	for(int i =0 ; i < 10; i++)
	{
		m_carVec.push_back(make_shared<Car>());
	}

    // ~Car() 10번 호출됨.
    // 이런식으로 delete를 직접해주지 않아도 된다.

	return 0;
}
```

```cpp
#include <iostream>
#include <memory>
#include <vector>
using namespace std;

class Car
{
    int color;
    int speed;
public:
    ~Car() { cout << "~Car()" << endl; }
 
    void Go() { cout << "Car go" << endl; }
};

void foo(Car* p)
{
    cout << "Delete Car" << endl;
}

int main() {
	vector<shared_ptr<Car>> m_carVec;

	for(int i =0 ; i < 10; i++)
	{
		m_carVec.push_back(make_shared<Car>());
	}

	m_carVec.pop_back();        // pop_back을 해도 역시하 소멸이 된다.

	cout << " New Line" << endl;

	return 0;
}
```

```cpp
#include <iostream>
#include <memory>       // for shared_ptr
using namespace std;

class Car
{
    int color;
    int speed;
public:
    ~Car() { cout << "~Car()" << endl; }
 
    void Go() { cout << "Car go" << endl; }
};

int main()
{
    int a = 0;      // 복사 생성자
    int a(0);       // direct 생성자

    shared_ptr<Car> p = new Car;        // error
    shared_ptr<Car> p(new Car);         // ok
}
```

shared_ptr은 복사 생성자로 생성이 불가능하다

```cpp
shared_ptr<Car> p1(new Car);
shared_ptr<Car> p2 = p1;
```

이렇게 여러 shared_ptr이 한 객체를 가리킬때는 두(p1, p2) 모두 사용을 종료할 경우만 메모리가 해제된다.

## 삭제자 넣기

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Car
{
    int color;
    int speed;
public:
    ~Car() { cout << "~Car()" << endl; }
 
    void Go() { cout << "Car go" << endl; }
};

void foo(Car* p)
{
    cout << "Delete Car" << endl;
    delete p;
}

int main()
{
    shared_ptr<Car> p(new Car, foo);    // 삭제자를 넣을 수 있다.

    shared_ptr<Car> p(new Car, [](Car* p){ delete p; }]);
}
```

```cpp
shared_ptr<Car> p(new Car, foo, MyAlloc<Car>);  // 할당자 역시 넣을 수 있다.
```

## 배열형태로 메모리를 할당한다면??

```cpp
#include <iostream>
#include <memory>       // for shared_ptr
using namespace std;

class Car
{
    int color;
    int speed;
public:
    ~Car() { cout << "~Car()" << endl; }
 
    void Go() { cout << "Car go" << endl; }
};

void foo(Car* p)
{
    cout << "Delete Car" << endl;
    delete p;
}

int main()
{
    shared_ptr<Car> p(new Car[10], [](Car* p){ delete[] p; });
    // 배열을 쓸 경우 삭제자를 무조건 넣어줘야한다.

    p[0].Go();      // error - shared_ptr은 배열을 고려해서 만들어지지 않았음
}
```

```cpp
// C++17 이후에는 shared_ptr에 배열을 지원한다.
shared_ptr<Car> p1(new Car[10]);       // 삭제자를 넣어줘야함!
shared_ptr<Car[]> p1(new Car[10]);      // ok!
```

```cpp
#include <iostream>
#include <memory>       // for shared_ptr
using namespace std;

class Car
{
    int color;
    int speed;
public:
    ~Car() { cout << "~Car()" << endl; }
 
    void Go() { cout << "Car go" << endl; }
};

int main()
{
    shared_ptr<Car> p(new Car);

    p->Go();
    //p.something; // shared_ptr자체 멤버에 접근
    p.get();                // 대상체의 포인터
    p.usecount();           // 참조계수 반환
    p.reset(new Car);       // 대상체 변경
    p.swap(new Car);        // 대상체 교환
}
```