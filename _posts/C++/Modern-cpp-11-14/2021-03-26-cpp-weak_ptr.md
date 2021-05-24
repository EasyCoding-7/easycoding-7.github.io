---
layout: post
title:  "(C++) weak_ptr"
summary: ""
author: C++
date: '2021-03-26 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
keywords: ['weak_ptr']
usemathjax: true
permalink: /blog/cpp/weak_ptr/
---

## 언제쓰나

* 참조변수가 증가하지 않는 스마트 포인터가 필요하다.
* 원본 객체가 파괴되었을 시 다 같이 삭제 되는 스마트 포인터가 필요하다.

---

## shared_ptr의 문제

```cpp
#include <iostream>
#include <string>
#include <memory>
using namespace std;

struct People
{
    People(string s) : name(s) {}
    ~People() { cout << "~People : " << name << endl; }

    string name;
    shared_ptr<People> bf;  // best friend
};

int main()
{
    shared_ptr<People> p1(new People("Kim"));
    shared_ptr<People> p2(new People("Lee"));

    p1->bf = p2;
    p2->bf = p1;
    // 자원이 파괴되지 않는다.
    // 서로가 서로를 가르키게 된다.
}
```

---

## 참조변수가 증가하지 않는 포인터가 필요하다 -> weak_ptr

```cpp
#include <iostream>
#include <string>
#include <memory>
using namespace std;

struct People
{
    People(string s) : name(s) {}
    ~People() { cout << "~People : " << name << endl; }

    string name;
    weak_ptr<People> bf;  
    // 참조변수가 증가하지 않는 포인터가 필요
    // 또한 원본 객체가 파괴되었는지 알수도 있다!!
};
```

---

## share_ptr 참조객체 개수 알아보기

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

int main()
{
    shared_ptr<Car> wp; // use count 증가
    shared_ptr<Car> sp(new Car);
    wp = sp;

    cout << sp.use_count() << endl; // 2
```

---

## weak_ptr 써보기

```cpp
int main()
{
    weak_ptr<Car> wp; // use count 증가 X
    
    {
        shared_ptr<Car> sp(new Car);
        wp = sp;

        cout << sp.use_count() << endl; // 1
    }

    if( wp.expired() )      // 원본 객체가 파괴되었는지 확인 가능
        cout << "destory" << endl;
    else
        cout << "not destory" << endl;
```

```cpp
int main()
{
    weak_ptr<Car> wp; // use count 증가 X
    
    //{
        shared_ptr<Car> sp(new Car);
        wp = sp;

        cout << sp.use_count() << endl; // 1
    //}

    if( wp.expired() )
        cout << "destory" << endl;
    else {
        cout << "not destory" << endl;
        wp->Go();       // error - weak_ptr은 대상 객체에 접근 불가
        // 이유는 use_count를 쓰지 않기에 다른곳에서 모르고 삭제를 해 버릴수 있다. 그런 실수를 방지하기 위해서 weak_ptr은 직접접근 불가

        // weak_ptr을 가지고 다시 shared_ptr을 만들어야한다.
    }
```

```cpp
shared_ptr<Car> sp2 = wp.lock();

if(sp2)
    sp->Go();
```

---

## Example

```cpp
class Dog {
    shared_ptr<Dog> m_pFriend;
public:
    string m_name;
    void bark() { cout << "Dog" << m_name << " rules!" << endl; }
    Dog() { //create }
    ~Dog() { cout << "dog is destroyed: " << m_name << endl; }
    void makeFriend(shared_ptr<Dog> f) { m_pFriend = f; }
};

int main()
{
    shared_ptr<Dog> pD(new Dog("Gunner"));
    shared_ptr<Dog> pD(new Dog("Smokey"));
    pD->makeFriend(pD2);
    pD->makeFriend(pD);
    // 서로 물고있어 삭제가 되지 않음.
    // cyclic reference
}
```

```cpp
class Dog {
    weak_ptr<Dog> m_pFriend;        // 해결
    // 사실 weak_ptr은 raw pointer(Dog *) 사용과 동일하다 단, weak_ptr로 delete를 할 수 없다는 점만 다르다
public:
    string m_name;
    void bark() { cout << "Dog" << m_name << " rules!" << endl; }
    Dog() { //create }
    ~Dog() { cout << "dog is destroyed: " << m_name << endl; }
    void makeFriend(shared_ptr<Dog> f) { m_pFriend = f; }
};
```

```cpp
// weak_ptr은 그냥 사용하면 안된다.
class Dog {
    weak_ptr<Dog> m_pFriend;
public:
    string m_name;
    void bark() { cout << "Dog" << m_name << " rules!" << endl; }
    Dog() { //create }
    ~Dog() { cout << "dog is destroyed: " << m_name << endl; }
    void makeFriend(shared_ptr<Dog> f) { m_pFriend = f; }
    void showFriend() {
        if(!m_pFriend.expired()) {
            cout << "My friend is : " << m_pFriend.lock()->m_name << endl;
            cout << m_pFriend.use_count() << endl;
        }
    }
};
```